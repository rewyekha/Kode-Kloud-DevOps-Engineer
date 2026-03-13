# Day 8: Attach Managed Disk to Azure Virtual Machine

The Nautilus DevOps team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing data disk to a virtual machine (VM).

An existing VM named `nautilus-vm` and a managed disk named `nautilus-disk` already exist in the East US region.

* Attach the disk `nautilus-disk` to the VM `nautilus-vm` as a data disk.
* Ensure the disk is attached to the VM `nautilus-vm`.

Make sure that the virtual machine initialization has been completed before submitting this task.\
Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

```

~ ➜  az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code EKWW84PCX to authenticate.

Retrieving tenants and subscriptions for the selection...

[Tenant and subscription selection]

No     Subscription name    Subscription ID                       Tenant
-----  -------------------  ------------------------------------  ----------------
[1] *  Azure Free Labs      f0c3bcdd-5ce2-4fa0-8cf3-41559747512b  azurefreekmlprod

The default is marked with an *; the default tenant is 'azurefreekmlprod' and subscription is 'Azure Free Labs' (f0c3bcdd-5ce2-4fa0-8cf3-41559747512b).

Select a subscription and tenant (Type a number or Enter for no changes): 1

Tenant: azurefreekmlprod
Subscription: Azure Free Labs (f0c3bcdd-5ce2-4fa0-8cf3-41559747512b)

[Announcements]
With the new Azure CLI login experience, you can select the subscription you want to use more easily. Learn more about it and its configuration at https://go.microsoft.com/fwlink/?linkid=2271236

If you encounter any problem, please open an issue at https://aka.ms/azclibug

[Warning] The login output has been updated. Please be aware that it no longer displays the full list of available subscriptions by default.


~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-597d6feef4614d08  westus      Succeeded

~ ➜  az vm list \
  --resource-group kml_rg_main-597d6feef4614d08 \
  --output table
Name         ResourceGroup                 Location    Zones
-----------  ----------------------------  ----------  -------
nautilus-vm  kml_rg_main-597d6feef4614d08  westus

~ ➜  az vm get-instance-view \
  --name nautilus-vm \
  --resource-group kml_rg_main-597d6feef4614d08 \
  --query "instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus" \
  --output table
Result
----------
VM running

~ ➜  az vm disk attach \
  --resource-group kml_rg_main-597d6feef4614d08 \
  --vm-name nautilus-vm \
  --name nautilus-disk

~ ➜  az vm show \
  --resource-group kml_rg_main-597d6feef4614d08 \
  --name nautilus-vm \
  --query "storageProfile.dataDisks[].name" \
  --output table
Result
-------------
nautilus-disk

~ ➜  az vm get-instance-view \
  --name nautilus-vm \
  --resource-group kml_rg_main-597d6feef4614d08 \
  --query "instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus" \
  --output table
Result
----------
VM running

~ ➜  
```

