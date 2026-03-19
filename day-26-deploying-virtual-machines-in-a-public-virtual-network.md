# Day 26: Deploying Virtual Machines in a Public Virtual Network

***

### Objective

The Nautilus DevOps Team was tasked to set up a **public-facing Azure Virtual Network (VNet)** to host resources accessible over the internet. A virtual machine (VM) was required to run public applications and allow SSH access for administration.

***

### Environment & Credentials

* **Azure Portal URL:** [https://portal.azure.com](https://portal.azure.com)
* **Username:** `kk_lab_user_main-80a1bb36624a4345@azurefreekmlprod.onmicrosoft.com`
* **Password:** `****`
* **Region:** East US
* **Resource Group:** `kml_rg_main-80a1bb36624a4345`

> **Note:** Resources were created in an existing resource group because the account does not have permission to create new groups.

***

### Solution Steps

#### 1. Define Variables

```bash
RESOURCE_GROUP="kml_rg_main-80a1bb36624a4345"
LOCATION="eastus"
VNET_NAME="datacenter-pub-vnet"
SUBNET_NAME="datacenter-pub-subnet"
VM_NAME="datacenter-pub-vm"
```

***

#### 2. Create Virtual Network & Subnet

```bash
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --address-prefix 10.0.0.0/16 \
  --subnet-name $SUBNET_NAME \
  --subnet-prefix 10.0.1.0/24 \
  --location $LOCATION
```

* Creates a **public VNet** (`datacenter-pub-vnet`)
* Creates a **subnet** (`datacenter-pub-subnet`) with auto-assigned public IP capability for attached resources

***

#### 3. Create VM with Standard SSD and Public IP

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --public-ip-sku Standard \
  --size Standard_B1s \
  --os-disk-size-gb 30 \
  --storage-sku StandardSSD_LRS \
  --location $LOCATION
```

**VM Specifications:**

* **OS:** Ubuntu 22.04
* **OS Disk:** 30 GB, Standard SSD
* **VM Size:** Standard\_B1s
* **Public IP:** Standard SKU
* **SSH Port:** 22 open to internet

***

#### 4. Open SSH Port

```bash
az vm open-port --resource-group $RESOURCE_GROUP --name $VM_NAME --port 22
```

***

#### 5. Verify Public IP

```bash
az vm list-ip-addresses --resource-group $RESOURCE_GROUP --name $VM_NAME --output table
```

**Example Output:**

| VirtualMachine    | PublicIPAddresses | PrivateIPAddresses |
| ----------------- | ----------------- | ------------------ |
| datacenter-pub-vm | 168.62.50.196     | 10.0.0.4           |

***

#### 6. Connect via SSH

```bash
ssh azureuser@168.62.50.196
```

> Successful login confirms SSH access is correctly configured.

***

### Results & Verification

* **Public VNet:** `datacenter-pub-vnet` ✅
* **Subnet:** `datacenter-pub-subnet` ✅
* **VM:** `datacenter-pub-vm` running with 30 GB Standard SSD ✅
* **Public IP:** 168.62.50.196 (Standard SKU) ✅
* **SSH Access:** Open and tested successfully ✅

***

### Notes

* Basic SKU public IPs are **not allowed** for this subscription, so **Standard SKU** was used.
* Resources are deployed in **East US region** only.
* The VM can now host public-facing applications for the Networking Team.

***

<figure><img src=".gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

