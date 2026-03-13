# Day 3: Create VM using Azure CLI

The Nautilus DevOps team is in the process of migrating some of their workloads to Azure. One of the tasks involves creating a new Virtual Machine (VM) using the Azure CLI. The team does not have access to the Azure portal but can manage Azure resources via the `azure-client` host (the landing host for this lab).

1\) Create a new Azure Virtual Machine named `xfusion-vm` using the Azure CLI.

2\) Use the `Ubuntu2204` image and set the VM size to `Standard_B2s`.

3\) Make sure the admin username is set to `azureuser` and SSH keys are generated for secure access.

4\) Use `Standard_LRS` storage account, disk size must be `30GB` and ensure the VM `xfusion-vm` is in the `running` state after creation.

## Create Azure Virtual Machine using Azure CLI

### Problem Statement

The Nautilus DevOps team is migrating workloads to Azure. As part of this task, a new Virtual Machine must be created using **Azure CLI only** (no Azure Portal access).

#### Requirements

1. Create a VM named **xfusion-vm** using Azure CLI
2. Use image **Ubuntu2204**
3. VM size must be **Standard\_B2s**
4. Admin username must be **azureuser**
5. SSH keys must be **generated automatically**
6. Storage account type must be **Standard\_LRS**
7. OS disk size must be **30 GB**
8. VM must be in **running state** after creation

***

### Step 1: Identify the Resource Group

List available resource groups:

```bash
az group list --output table
```

Note down the resource group name (for example: `kodekloud-rg`).

***

### Step 2: Create the Virtual Machine

Run the following command (replace `<RESOURCE_GROUP>` with the actual name):

```bash
az vm create \
  --resource-group <RESOURCE_GROUP> \
  --name xfusion-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30
```

#### What this command does

* Creates VM **xfusion-vm**
* Uses **Ubuntu 22.04 LTS** image
* Sets VM size to **Standard\_B2s**
* Creates user **azureuser**
* Auto-generates SSH key pair
* Uses **Standard\_LRS** managed disk
* Sets OS disk size to **30 GB**

***

### Step 3: Verify VM Power State

Check VM status:

```bash
az vm get-instance-view \
  --resource-group <RESOURCE_GROUP> \
  --name xfusion-vm \
  --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
  --output table
```

Expected output:

```
VM running
```

***

### Faced Issue & Solution (From Lab Discussion)

#### ❌ Issue Faced

* Running `az login` redirected to a browser
* Asked for user email and password
* Lab-provided credentials were no longer available

#### ✅ Solution

> You **do not need to run `az login`** in this lab. The terminal session is **already authenticated**.

All you need is:

1. A valid resource group (`az group list`)
2. Correct Azure CLI command to create the VM

Once this is done, VM creation works without any login issues.



```

~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-c00ae911ea294054  westus      Succeeded

~ ➜  az vm create \
--resource-group kml_rg_main-c00ae911ea294054 \
--name xfusion-vm \
--image Ubuntu2204 \
--size Standard_B2s \
--admin-username azureuser \
--generate-ssh-keys \
--storage-sku Standard_LRS \
--os-disk-size-gb 30
SSH key files '/root/.ssh/id_rsa' and '/root/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage, back up your keys to a safe location.
{
  "fqdns": "",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-c00ae911ea294054/providers/Microsoft.Compute/virtualMachines/xfusion-vm",
  "location": "westus",
  "macAddress": "60-45-BD-02-40-51",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "172.184.234.149",
  "resourceGroup": "kml_rg_main-c00ae911ea294054",
  "zones": ""
}

~ ➜  az vm get-instance-view \
--resource-group <RESOURCE_GROUP> \
--name xfusion-vm \
--query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
--output table
-bash: RESOURCE_GROUP: No such file or directory

~ ✖ az vm get-instance-view \
  --resource-group kml_rg_main-c00ae911ea294054 \
  --name xfusion-vm \
  --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
  --output table
Result
----------
VM running

~ ➜  
```
