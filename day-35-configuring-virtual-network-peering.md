# Day 35: Configuring Virtual Network Peering

## Azure VNet Peering Demo – Public VM to Private VM Connectivity

### 📌 Objective

The Nautilus DevOps team needs to demonstrate **Azure VNet Peering** to allow communication between two Virtual Networks.

One VNet contains a **public VM**, while the other contains a **private VM**.

After configuring VNet Peering, the **public VM should be able to ping the private VM using its private IP address**.

***

## 📝 Problem Statement

The Nautilus DevOps team has been tasked with demonstrating the use of VNet Peering to enable communication between two VNets.

#### Existing Azure Resources

| Resource       | Name                 |
| -------------- | -------------------- |
| Public VM      | `devops-pub-vm`      |
| Private VNet   | `devops-priv-vnet`   |
| Private Subnet | `devops-priv-subnet` |
| Private VM     | `devops-priv-vm`     |

#### Tasks

1. Create **VNet Peering** between the Public VNet and Private VNet.
2.  Peering Name:

    ```
    devops-pub-to-priv-peering
    ```
3. SSH into the public VM and verify connectivity by **pinging the private VM**.

#### Notes

* Create resources in **East US region**
* Use the provided Azure credentials.

***

## 🔎 Step 1 — Verify Resource Group

First, confirm the available resource groups.

```bash
az group list
```

#### Output

```bash
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-ffbbb82637b84a2c",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-ffbbb82637b84a2c",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

#### Explanation

This confirms that the resource group:

```
kml_rg_main-ffbbb82637b84a2c
```

exists in the **East US region** and will be used for all networking resources.

***

## 🔎 Step 2 — Verify Existing VNets

Next, check the available Virtual Networks.

```bash
az network vnet list -g kml_rg_main-ffbbb82637b84a2c -o table
```

#### Output

```bash
Name              ResourceGroup                 Location    NumSubnets    Prefixes
----------------  ----------------------------  ----------  ------------  -----------
devops-priv-vnet  kml_rg_main-ffbbb82637b84a2c  eastus      1             10.1.0.0/16
devops-pub-vnet   kml_rg_main-ffbbb82637b84a2c  eastus      1             10.2.0.0/16
```

#### Explanation

Two VNets already exist:

| VNet             | Address Space |
| ---------------- | ------------- |
| devops-priv-vnet | 10.1.0.0/16   |
| devops-pub-vnet  | 10.2.0.0/16   |

These networks **do not overlap**, which is required for **VNet Peering**.

***

## 🔗 Step 3 — Create VNet Peering (Public → Private)

Now create a peering connection from the **public VNet to the private VNet**.

```bash
az network vnet peering create \
--name devops-pub-to-priv-peering \
--resource-group kml_rg_main-ffbbb82637b84a2c \
--vnet-name devops-pub-vnet \
--remote-vnet devops-priv-vnet \
--allow-vnet-access
```

#### Output

```json
{
  "name": "devops-pub-to-priv-peering",
  "peeringState": "Initiated",
  "provisioningState": "Succeeded",
  "remoteAddressSpace": {
    "addressPrefixes": [
      "10.1.0.0/16"
    ]
  }
}
```

#### Explanation

This command creates a **one-way VNet Peering** from:

```
devops-pub-vnet → devops-priv-vnet
```

The state is initially **Initiated** because the reverse peering has not yet been created.

***

## 🔗 Step 4 — Create Reverse Peering (Private → Public)

Azure VNet Peering must be configured **in both directions**.

```bash
az network vnet peering create \
--name devops-priv-to-pub-peering \
--resource-group kml_rg_main-ffbbb82637b84a2c \
--vnet-name devops-priv-vnet \
--remote-vnet devops-pub-vnet \
--allow-vnet-access
```

#### Output

```json
{
  "name": "devops-priv-to-pub-peering",
  "peeringState": "Connected",
  "provisioningState": "Succeeded"
}
```

#### Explanation

This establishes the return connection:

```
devops-priv-vnet → devops-pub-vnet
```

Once both sides exist, the peering state becomes **Connected**.

***

## 🔎 Step 5 — Verify Peering Status

```bash
az network vnet peering list \
--resource-group kml_rg_main-ffbbb82637b84a2c \
--vnet-name devops-pub-vnet \
-o table
```

#### Output

```bash
Name                        PeeringState    ProvisioningState
--------------------------  --------------  -----------------
devops-pub-to-priv-peering  Connected       Succeeded
```

#### Explanation

The **Connected** state confirms that the VNet Peering is active and traffic can flow between networks.

***

## 🌐 Step 6 — Retrieve VM IP Addresses

### Public VM

```bash
az vm list-ip-addresses \
-n devops-pub-vm \
-g kml_rg_main-ffbbb82637b84a2c \
-o table
```

#### Output

```bash
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
devops-pub-vm     20.127.42.246        10.2.1.4
```

### Private VM

```bash
az vm list-ip-addresses \
-n devops-priv-vm \
-g kml_rg_main-ffbbb82637b84a2c \
-o table
```

#### Output

```bash
VirtualMachine    PrivateIPAddresses
----------------  --------------------
devops-priv-vm    10.1.1.4
```

#### Explanation

| VM         | IP Type    | IP            |
| ---------- | ---------- | ------------- |
| Public VM  | Public IP  | 20.127.42.246 |
| Public VM  | Private IP | 10.2.1.4      |
| Private VM | Private IP | 10.1.1.4      |

The **private VM only has a private IP**, so communication must occur through **VNet Peering**.

***

## 🔐 Step 7 — SSH into Public VM

```bash
ssh azureuser@20.127.42.246
```

#### Output

```bash
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)
```

#### Explanation

We connect to the **public VM**, which will be used to test connectivity to the **private VM**.

***

## 📡 Step 8 — Test Connectivity

Ping the private VM.

```bash
ping 10.1.1.4
```

#### Output

```bash
PING 10.1.1.4 (10.1.1.4) 56(84) bytes of data.
64 bytes from 10.1.1.4: icmp_seq=1 ttl=64 time=2.69 ms
64 bytes from 10.1.1.4: icmp_seq=2 ttl=64 time=1.58 ms
64 bytes from 10.1.1.4: icmp_seq=3 ttl=64 time=1.32 ms
64 bytes from 10.1.1.4: icmp_seq=4 ttl=64 time=1.29 ms
```

#### Statistics

```bash
18 packets transmitted, 18 received, 0% packet loss
```

#### Explanation

Successful ping responses confirm that:

* **VNet Peering is working**
* Traffic flows between **devops-pub-vnet and devops-priv-vnet**
* The **public VM can communicate with the private VM**

***

## ✅ Final Architecture

```
Public VM
(devops-pub-vm)
20.127.42.246
10.2.1.4
      │
      │  VNet Peering
      ▼
Public VNet (10.2.0.0/16)
      │
      │
Private VNet (10.1.0.0/16)
      │
      ▼
Private VM
(devops-priv-vm)
10.1.1.4
```

***

## 🎯 Key Takeaways

✔ VNet Peering connects two Azure VNets privately\
✔ Address spaces must **not overlap**\
✔ Peering must exist **in both directions**\
✔ Communication occurs via **private IP addresses**

***

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>



Below is a **GitBook-style documentation for Azure Portal (GUI) steps**. You can place this as a **separate section in your GitBook** like:

`Azure → Networking → VNet Peering (Portal Method)` 📘

***

## Azure VNet Peering Using Azure Portal (GUI)

### 📌 Objective

Demonstrate **Azure Virtual Network Peering** to allow communication between:

* A **Public Virtual Network** containing a public VM
* A **Private Virtual Network** containing a private VM

After configuring VNet Peering, we verify connectivity by **pinging the private VM from the public VM**.

***

## 📝 Problem Statement

The Nautilus DevOps team has been tasked with demonstrating the use of **VNet Peering** to enable communication between two VNets.

### Existing Azure Resources

| Resource       | Name                 |
| -------------- | -------------------- |
| Public VM      | `devops-pub-vm`      |
| Private VNet   | `devops-priv-vnet`   |
| Private Subnet | `devops-priv-subnet` |
| Private VM     | `devops-priv-vm`     |

### Requirements

1. Create **VNet Peering**
2. Peering Name

```
devops-pub-to-priv-peering
```

3. Verify connectivity using **SSH + Ping**

***

## 🌐 Step 1 — Login to Azure Portal

Open the Azure Portal:

🔗 [https://portal.azure.com](https://portal.azure.com)

Login using the provided credentials.

| Field    | Value                                                                                                                                              |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username | [kk\_lab\_user\_main-ffbbb82637b84a2c@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-ffbbb82637b84a2c@azurefreekmlprod.onmicrosoft.com) |
| Password | Provided in lab                                                                                                                                    |

After login, you will reach the **Azure Dashboard**.

***

## 🔎 Step 2 — Locate Virtual Networks

1. In the search bar type:

```
Virtual networks
```

2. Click **Virtual Networks**

You should see two VNets:

| VNet               | Address Space |
| ------------------ | ------------- |
| `devops-pub-vnet`  | 10.2.0.0/16   |
| `devops-priv-vnet` | 10.1.0.0/16   |

These VNets are located in:

```
East US
```

***

## 🔗 Step 3 — Create Peering from Public VNet

1. Click

```
devops-pub-vnet
```

2. In the left navigation panel select:

```
Peerings
```

3. Click

```
+ Add
```

Fill the peering configuration.

#### Basic Configuration

| Setting                      | Value                        |
| ---------------------------- | ---------------------------- |
| Peering link name            | `devops-pub-to-priv-peering` |
| Remote virtual network       | `devops-priv-vnet`           |
| Allow virtual network access | Enabled                      |

#### Remote Peering Configuration

| Setting                      | Value                        |
| ---------------------------- | ---------------------------- |
| Peering link name            | `devops-priv-to-pub-peering` |
| Allow virtual network access | Enabled                      |

Then click:

```
Add
```

***

## 🔗 Step 4 — Verify Peering

After creation, you should see:

| Peering Name               | State     |
| -------------------------- | --------- |
| devops-pub-to-priv-peering | Connected |
| devops-priv-to-pub-peering | Connected |

This means the two VNets are successfully connected.

***

## 🖥 Step 5 — Locate Public VM

Search in the portal:

```
Virtual Machines
```

Click:

```
devops-pub-vm
```

From the **Overview page**, note:

| Property   | Value         |
| ---------- | ------------- |
| Public IP  | 20.127.42.246 |
| Private IP | 10.2.1.4      |

***

## 🖥 Step 6 — Locate Private VM

Open:

```
devops-priv-vm
```

Check the **Networking section**.

Private IP should be:

```
10.1.1.4
```

This VM **does not have a public IP**, so access must occur through the peered network.

***

## 🔐 Step 7 — SSH into Public VM

Use a terminal.

```bash
ssh azureuser@20.127.42.246
```

Example output:

```bash
Welcome to Ubuntu 22.04.5 LTS
```

You are now connected to the **public VM**.

***

## 📡 Step 8 — Test Network Connectivity

Ping the private VM from the public VM.

```bash
ping 10.1.1.4
```

#### Example Output

```bash
PING 10.1.1.4 (10.1.1.4) 56(84) bytes of data.
64 bytes from 10.1.1.4: icmp_seq=1 ttl=64 time=2.69 ms
64 bytes from 10.1.1.4: icmp_seq=2 ttl=64 time=1.58 ms
64 bytes from 10.1.1.4: icmp_seq=3 ttl=64 time=1.32 ms
```

Ping statistics:

```bash
18 packets transmitted, 18 received, 0% packet loss
```

***

## 🏗 Final Architecture

```
Public VM
(devops-pub-vm)
20.127.42.246
10.2.1.4
        │
        │ VNet Peering
        ▼
Public VNet
10.2.0.0/16
        │
        │
Private VNet
10.1.0.0/16
        ▼
Private VM
(devops-priv-vm)
10.1.1.4
```

***

## ✅ Result

The following tasks were successfully completed:

✔ VNet Peering created\
✔ Public and Private VNets connected\
✔ SSH access to Public VM\
✔ Successful ping to Private VM

This confirms **private communication between VNets using Azure VNet Peering**.

***

## 🎯 Key Concepts

| Concept                  | Description                           |
| ------------------------ | ------------------------------------- |
| VNet Peering             | Connects two Azure VNets privately    |
| Non-overlapping CIDR     | Required for peering                  |
| Private IP Communication | Traffic flows using private addresses |
| Bidirectional Peering    | Required for full connectivity        |

***

✅ **CLI Method:** Faster for automation\
✅ **Portal Method:** Easier for beginners and visualization

***

**Common Error Scenarios:**

* Ping fails after peering
* NSG blocking ICMP
* Peering stuck in Initiated
* Address space overlap

***

## ⚠️ Troubleshooting Azure VNet Peering Issues

Even after creating VNet Peering, connectivity might fail. This section explains common issues and their fixes.

***

### 1️⃣ Ping Fails After Peering

#### Symptom

* VNet peering shows **Connected**
* VM in Public VNet **cannot ping** VM in Private VNet

#### Possible Causes

* **ICMP is blocked** (by NSG or firewall)
* VM uses **wrong private IP**
* Subnet **route or gateway conflict**

#### Steps to Fix

1. **Check VM private IPs**

```bash
az vm list-ip-addresses -n devops-priv-vm -g <resource-group> -o table
```

2. **Ping private IP from public VM**

```bash
ping <private-ip>
```

3. **Check Network Security Group (NSG) rules**

* Go to **Virtual Network → Subnet → NSG**
* Ensure **Inbound Rule allows ICMP / All Traffic** for the peered subnet

4. **Check routing**

* Make sure **subnet does not have custom routes** blocking traffic

***

### 2️⃣ NSG Blocking ICMP

#### Symptom

* VMs are pingable internally but **ping fails between VNets**
* Peering is connected

#### Cause

* Network Security Group attached to **VM NIC or subnet** blocks ICMP (protocol 1)

#### Steps to Fix

1. Go to **Azure Portal → Network Security Groups**
2. Click **Inbound security rules**
3. Add a rule:

| Setting                 | Value                    |
| ----------------------- | ------------------------ |
| Source                  | Any (or specific subnet) |
| Source port ranges      | \*                       |
| Destination             | Any (or specific subnet) |
| Destination port ranges | \*                       |
| Protocol                | ICMP                     |
| Action                  | Allow                    |
| Priority                | 100-200                  |

4. Save changes and retry ping

***

### 3️⃣ Peering Stuck in “Initiated”

#### Symptom

* Peering state shows:

```
PeeringState: Initiated
```

* Traffic cannot flow

#### Cause

* **Reverse peering not created**
* Peering requires **both VNets to have a peering object**

#### Steps to Fix

1. Create the **reverse peering**:

```bash
az network vnet peering create \
--name <private-to-public-peering> \
--resource-group <rg> \
--vnet-name <private-vnet> \
--remote-vnet <public-vnet> \
--allow-vnet-access
```

2. Verify status:

```bash
az network vnet peering list -g <rg> -n <public-vnet> -o table
```

* Peering state should now show **Connected**

***

### 4️⃣ Address Space Overlap

#### Symptom

* VNet peering creation fails
* Error message:

```
The remote virtual network address space overlaps with this virtual network.
```

#### Cause

* **Both VNets have overlapping CIDR ranges**
* Azure does not allow peering for overlapping IP ranges

#### Steps to Fix

1. Check VNet address space:

```bash
az network vnet show -n <vnet-name> -g <rg> --query addressSpace.addressPrefixes
```

2. Ensure **CIDR ranges do not overlap**, e.g.:

| VNet             | Address Space |
| ---------------- | ------------- |
| devops-priv-vnet | 10.1.0.0/16   |
| devops-pub-vnet  | 10.2.0.0/16   |

3. If overlapping, create a **new VNet** with non-overlapping CIDR:

```bash
az network vnet create \
--name devops-pub-vnet2 \
--resource-group <rg> \
--address-prefix 10.3.0.0/16 \
--subnet-name devops-pub-subnet
```

4. Recreate VNet peering

***

## 💡 Tips for Troubleshooting

| Issue               | Quick Check                        |
| ------------------- | ---------------------------------- |
| Ping fails          | ICMP allowed, correct private IP   |
| NSG blocks traffic  | Check inbound rules on subnet/NIC  |
| Peering not working | Verify **both VNets have peering** |
| Overlapping CIDR    | Check VNet address space           |

***

✅ **Pro Tip**

* Always test **ping after peering**
* Use **Azure Portal → Network Watcher → IP Flow Verify** for deeper analysis

