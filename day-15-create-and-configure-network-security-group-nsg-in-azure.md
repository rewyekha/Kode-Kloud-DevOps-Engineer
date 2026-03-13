# Day 15: Create and Configure Network Security Group (NSG) in Azure



The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a network security group (NSG) with the following requirements:

* Name of the NSG should be `xfusion-nsg`.
* Add an inbound security rule named `Allow-HTTP` for `HTTP` service on port `80`, with the source CIDR range of `0.0.0.0/0`.
* Add another inbound security rule named `Allow-SSH` for `SSH` service on port `22`, with the source CIDR range of `0.0.0.0/0`.

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-f9737f435f6d4e2d@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-f9737f435f6d4e2d@azurefreekmlprod.onmicrosoft.com) |
| Password   | #6Gu4cHE                                                                                                                                           |
| Start Time | Mon Jan 12 16:45:29 UTC 2026                                                                                                                       |
| End Time   | Mon Jan 12 17:45:29 UTC 2026                                                                                                                       |

```bash
~ ➜  az network nsg rule list \
  --resource-group kml_rg_main-f9737f435f6d4e2d \
  --nsg-name xfusion-nsg \
  --query "[].{Name:name,Port:destinationPortRange,Source:sourceAddressPrefix,Access:access,Priority:priority}" \
  --output table
Name        Port    Source    Access    Priority
----------  ------  --------  --------  ----------
Allow-HTTP  80      *         Allow     100
Allow-SSH   22      *         Allow     110

~ ➜  
```

### Step 1: Sign in to Azure Portal

1. Open **Azure Portal**\
   👉 [https://portal.azure.com](https://portal.azure.com)
2. Sign in using:
   * **Username:**\
     `kk_lab_user_main-f9737f435f6d4e2d@azurefreekmlprod.onmicrosoft.com`
   * **Password:**\
     `#6Gu4cHE`

***

### Step 2: Navigate to Network Security Groups

1. In the Azure Portal search bar (top), type **Network security groups**
2. Click **Network security groups** from the results

***

### Step 3: Create the Network Security Group

1. Click **+ Create**
2. Fill in the **Basics** tab:
   * **Subscription:** (leave default)
   * **Resource group:** select an existing resource group\
     &#xNAN;_(or create a new one if required)_
   * **Name:** `xfusion-nsg`
   * **Region:** choose the required region (same region as your resources)
3. Click **Review + create**
4. Click **Create**

✅ NSG `xfusion-nsg` is now created.

***

### Step 4: Add Inbound Rule for HTTP (Port 80)

1. Open the newly created **xfusion-nsg**
2. In the left menu, click **Inbound security rules**
3. Click **+ Add**

Fill in the rule details:

* **Source:** IP Addresses
* **Source IP addresses/CIDR ranges:** `0.0.0.0/0`
* **Source port ranges:** `*`
* **Destination:** Any
* **Service:** HTTP
* **Destination port ranges:** `80`
* **Protocol:** TCP
* **Action:** Allow
* **Priority:** `100`
* **Name:** `Allow-HTTP`

4. Click **Add**

***

### Step 5: Add Inbound Rule for SSH (Port 22)

1. Still under **Inbound security rules**, click **+ Add**

Fill in the rule details:

* **Source:** IP Addresses
* **Source IP addresses/CIDR ranges:** `0.0.0.0/0`
* **Source port ranges:** `*`
* **Destination:** Any
* **Service:** SSH
* **Destination port ranges:** `22`
* **Protocol:** TCP
* **Action:** Allow
* **Priority:** `110`
* **Name:** `Allow-SSH`

2. Click **Add**

***

### Step 6: Verify Rules

Under **Inbound security rules**, confirm:

* ✅ **Allow-HTTP** → Port **80**, Source **0.0.0.0/0**
* ✅ **Allow-SSH** → Port **22**, Source **0.0.0.0/0**

***

#### 🎯 Final Result

* Network Security Group **xfusion-nsg** created
* HTTP (80) and SSH (22) inbound access allowed from **anywhere**

