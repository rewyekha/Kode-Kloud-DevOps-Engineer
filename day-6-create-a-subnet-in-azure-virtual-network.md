# Day 6: Create a Subnet in Azure Virtual Network

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition.

For this task, create a Virtual Network (VNet) named `datacenter-vnet` and one subnet named `datacenter-subnet` within the VNet in the `East US` region. Make sure the `IPv4 address range` is `10.0.0.0/16`.

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-4bc24aa297d14928@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-4bc24aa297d14928@azurefreekmlprod.onmicrosoft.com) |
| Password   | \&AedCxB^                                                                                                                                          |

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### ✅ Task Requirements (Recap)

* **VNet name:** `datacenter-vnet`
* **Subnet name:** `datacenter-subnet`
* **Region:** East US
* **IPv4 address range:** `10.0.0.0/16`
* Use **Azure CLI**
* Authenticate using credentials from `showcreds`

***

### 🔹 Step 1: Retrieve Azure Credentials

On the **azure-client host**, run:

```bash
showcreds
```

Note down:

* `clientId`
* `clientSecret`
* `tenantId`
* `subscriptionId`

***

### 🔹 Step 2: Login to Azure (Service Principal)

```bash
az login --service-principal \
  --username <CLIENT_ID> \
  --password <CLIENT_SECRET> \
  --tenant <TENANT_ID>
```

Set the subscription:

```bash
az account set --subscription <SUBSCRIPTION_ID>
```

***

### 🔹 Step 3: Identify the Existing Resource Group

```bash
az group list --output table
```

👉 Use the **existing resource group** shown (do **not** create a new one unless asked).

Example:

```
kml_rg_main-xxxxxxxxxxxx
```

***

### 🔹 Step 4: Create VNet **and** Subnet (Single Command)

```bash
az network vnet create \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vnet \
  --location eastus \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name datacenter-subnet \
  --subnet-prefixes 10.0.0.0/24
```

#### Why this works

* VNet address space: `10.0.0.0/16` ✅ (required)
* Subnet created inside the VNet
* Subnet CIDR (`10.0.0.0/24`) is valid and inside `/16`

***

### 🔹 Step 5: Verify VNet and Subnet

#### Verify VNet

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vnet \
  --query "{Name:name,Location:location,AddressSpace:addressSpace.addressPrefixes}" \
  --output table
```

#### Verify Subnet

```bash
az network vnet subnet show \
  --resource-group <RESOURCE_GROUP> \
  --vnet-name datacenter-vnet \
  --name datacenter-subnet \
  --output table
```

***

### ✅ Final Checklist (Evaluator Looks For)

✔ VNet named **datacenter-vnet**\
✔ Subnet named **datacenter-subnet**\
✔ Region is **East US**\
✔ VNet IPv4 range is **10.0.0.0/16**\
✔ Created using Azure CLI with provided credentials

***

### 🧠 KodeKloud Tip

> If a task asks for **VNet + subnet**, using a **single `az network vnet create` command** is preferred and clean.

