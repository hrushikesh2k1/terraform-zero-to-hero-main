⚙️ Step 1: Create a Shell Script to Run Terraform Drift Check

Create a new bash script, for example /opt/terraform/scripts/terraform-drift-check.sh

#!/bin/bash

# Define working directory where Terraform code lives
WORKDIR="/opt/terraform/azure"

# Go to Terraform project directory
cd $WORKDIR || exit

# Run Terraform refresh and store output in a log file
/usr/local/bin/terraform refresh -no-color > drift_check.log 2>&1

# Optional: Check if log file contains any drift messages
if grep -q "changed" drift_check.log; then
    echo "[$(date)] ⚠️ Drift detected in Terraform state!" >> /var/log/terraform_drift.log
    # You can integrate alert here, e.g., send Slack or Teams notification
else
    echo "[$(date)] ✅ No drift detected." >> /var/log/terraform_drift.log
fi

💡 Notes:

terraform refresh → syncs Terraform state with real Azure.

-no-color → removes ANSI color codes for clean logs.

2>&1 → sends both standard output and errors to the same file.

grep -q → quietly checks if a word exists in the log.

⚙️ Step 2: Give Permissions and Test It Manually
sudo chmod +x /opt/terraform/scripts/terraform-drift-check.sh
sudo /opt/terraform/scripts/terraform-drift-check.sh


Check if logs are generated:

cat /opt/terraform/azure/drift_check.log
cat /var/log/terraform_drift.log


✅ If you see “No drift detected” → it’s working fine.

⚙️ Step 3: Schedule It Using a Cron Job

Now we’ll tell Linux to automatically run this script at regular intervals using cron.

Open the cron configuration:

crontab -e


Then add this line at the bottom:

0 * * * * /opt/terraform/scripts/terraform-drift-check.sh


This means:

0 * * * * → run every hour (at 00 minutes)

/opt/terraform/scripts/terraform-drift-check.sh → full path to your script






*    *    *    *    *    <command to run>
|    |    |    |    |
|    |    |    |    └─── Day of the week (0 - 6) [Sunday = 0]
|    |    |    └──────── Month (1 - 12)
|    |    └───────────── Day of the month (1 - 31)
|    └────────────────── Hour (0 - 23)
└─────────────────────── Minute (0 - 59)
