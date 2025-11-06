**Scenario:**

Your organization initially created Azure infrastructure (like Virtual Machines, VNets, Storage Accounts) manually from the Azure Portal or through ARM templates.

Now, your team wants to move to Terraform for better Infrastructure as Code (IaC) management.
You’ve been asked to import the existing Azure resources into Terraform without recreating them.

**Approach:**

Terraform can take ownership of existing Azure resources by using:

The terraform import command

Or, for Terraform 1.5+, the import block

And the terraform plan -generate-config-out command to generate resource configuration automatically

🧩 Step-by-Step Implementation
1️⃣ Create a new Terraform project directory
mkdir terraform-azure-import && cd terraform-azure-import

2️⃣ Create a main Terraform configuration file (main.tf)
provider "azurerm" {
  features {}
}

# Import block (Terraform v1.5+ feature)
import {
  to = azurerm_virtual_machine.example
  id = "/subscriptions/<sub-id>/resourceGroups/demo-rg/providers/Microsoft.Compute/virtualMachines/demo-vm"
}


💡 The id above is the Azure Resource ID, which you can find using:

az resource show -g demo-rg -n demo-vm --resource-type Microsoft.Compute/virtualMachines --query id -o tsv

3️⃣ Initialize Terraform
terraform init


This downloads the AzureRM provider plugin and prepares your working directory.

4️⃣ Generate Terraform configuration automatically
terraform plan -generate-config-out=generated.tf


Terraform will create a new configuration file (e.g., generated.tf) that contains the HCL representation of your imported resource.

Example output:

resource "azurerm_virtual_machine" "example" {
  name                  = "demo-vm"
  location              = "eastus"
  resource_group_name   = "demo-rg"
  network_interface_ids = ["/subscriptions/<sub-id>/resourceGroups/demo-rg/providers/Microsoft.Network/networkInterfaces/demo-nic"]
  vm_size               = "Standard_B1s"

  storage_os_disk {
    name              = "demo-osdisk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  os_profile {
    computer_name  = "demo-vm"
    admin_username = "azureuser"
  }
}

5️⃣ Cleanup and Refactor

You can now:

Copy only the required attributes (like vm_size, os_profile)

Remove unnecessary auto-generated ones (like time stamps)

Add variables for reusable parameters (e.g., region, VM size)

6️⃣ Import the resource into Terraform state

Run the import command manually:

terraform import azurerm_virtual_machine.example /subscriptions/<sub-id>/resourceGroups/demo-rg/providers/Microsoft.Compute/virtualMachines/demo-vm


This creates a terraform.tfstate file that maps this existing Azure VM to your Terraform configuration.

7️⃣ Verify
terraform plan


If you get:

No changes. Infrastructure is up-to-date.


✅ Terraform has successfully imported and now manages that Azure resource.

Challenges Faced
Challenge	Explanation
Complex Dependencies	Importing dependent resources like NICs, Disks, NSGs separately
State Conflicts	Conflicts when multiple people import resources simultaneously
Missing Fields	Terraform expects explicit fields even if Azure sets defaults
Resource Count	Tedious when migrating large-scale environments
