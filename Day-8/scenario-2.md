**Scenario:**

All your infrastructure (VMs, VNets, Storage, AKS, etc.) is managed via Terraform.
However, someone from the operations team made manual changes (for example, changing the VM size or NSG rules) directly through the Azure Portal.

Terraform’s local state file won’t automatically detect these manual changes.
You need a mechanism to detect drift and alert the DevOps team.

**Approach 1 — Using Terraform Refresh Command**

Terraform provides the terraform refresh command, which syncs your local state with the real Azure configuration.

1️⃣ Run Terraform Refresh
terraform refresh


This updates your Terraform state (terraform.tfstate) to reflect any manual modifications in Azure.

2️⃣ Automate with Cron Job

You can schedule this command to run periodically for drift detection:

crontab -e


Add this line:

0 * * * * cd /opt/terraform/azure && terraform refresh -no-color > drift_check.log


This runs every hour, logs output, and helps detect configuration drifts.

3️⃣ Send Notifications (Optional)

You can integrate this with Azure Monitor or send alerts via email or Teams webhook if the drift_check.log shows differences.

Approach 2 — Using Azure Activity Logs + Azure Function

Terraform doesn’t auto-detect drift, but Azure provides Activity Logs that record all resource-level actions.

1️⃣ Enable Activity Logs for all subscriptions

Go to Azure Monitor → Activity Logs

Capture all “Write” and “Update” operations

2️⃣ Send logs to a Log Analytics workspace

3️⃣ Use Azure Function (Python) to detect manual changes:

import json
import requests

def main(event: str):
    data = json.loads(event)
    user = data['claims']['aud'] if 'claims' in data else "Unknown"
    operation = data['operationName']

    if "terraform" not in user.lower():
        message = f"⚠️ Manual change detected: {operation} by {user}"
        webhook_url = "https://outlook.office.com/webhook/your-teams-webhook"
        requests.post(webhook_url, json={"text": message})


4️⃣ Notify via Microsoft Teams or Azure Monitor Alerts

This approach gives near real-time alerts whenever a manual change is made outside Terraform.

Approach 3 — Restrict Portal-Level Access with Azure RBAC

Prevent manual changes by enforcing least privilege:

Example — Deny role modification or resource update access:

{
  "RoleName": "TerraformOperator",
  "Description": "Allows deployment via Terraform only",
  "Actions": [
    "Microsoft.Resources/subscriptions/resourceGroups/*/read",
    "Microsoft.Resources/deployments/*"
  ],
  "NotActions": [
    "Microsoft.Compute/virtualMachines/write",
    "Microsoft.Network/networkSecurityGroups/write"
  ]
}

Challenges Faced
Challenge	Description
Manual Edits	Portal edits cause state drift
Lack of Audit	Terraform doesn’t auto-track manual actions
Access Issues	Need proper RBAC controls to prevent changes
Notification Delay	Activity logs sometimes take a few minutes to sync
✅ Interview Summary Answer

“In one instance, a teammate changed VM size manually from the Azure Portal.
Terraform wasn’t aware of the change, so we implemented automated drift detection.
We scheduled terraform refresh jobs and integrated Azure Activity Logs with Azure Functions to send alerts for non-Terraform user updates.
We also restricted portal write access using custom RBAC roles.
This helped maintain IaC consistency and prevented future unauthorized changes.”
