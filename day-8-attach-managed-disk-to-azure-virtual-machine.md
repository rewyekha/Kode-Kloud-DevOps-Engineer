# Day 8: Attach Managed Disk to Azure Virtual Machine

The Nautilus DevOps team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing data disk to a virtual machine (VM).

An existing VM named `nautilus-vm` and a managed disk named `nautilus-disk` already exist in the East US region.

* Attach the disk `nautilus-disk` to the VM `nautilus-vm` as a data disk.
* Ensure the disk is attached to the VM `nautilus-vm`.

Make sure that the virtual machine initialization has been completed before submitting this task.\
Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

```bash

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

