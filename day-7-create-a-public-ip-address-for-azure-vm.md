# Day 7: Create a Public IP Address for Azure VM

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, allocate a `Public IP` address, name it as `devops-pip`.

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-4016c8d824df43f3@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-4016c8d824df43f3@azurefreekmlprod.onmicrosoft.com) |
| Password   | DN57BaKT                                                                                                                                           |

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>



### Option 1: Using Azure CLI (Recommended for DevOps)

On the **azure-client** host (where `showcreds` works), run:

```bash
az login
```

Then create the Public IP address.

> You **must know**:

* Resource Group name
* Azure region (location)

#### Command

```bash
az network public-ip create \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name devops-pip \
  --sku Standard \
  --allocation-method Static \
  --location <AZURE_REGION>
```

#### Example

```bash
az network public-ip create \
  --resource-group devops-rg \
  --name devops-pip \
  --sku Standard \
  --allocation-method Static \
  --location eastus
```

#### Verify

```bash
az network public-ip show \
  --resource-group devops-rg \
  --name devops-pip \
  --output table
```

***

### Option 2: Using Azure Portal (GUI)

1. Go to [**https://portal.azure.com**](https://portal.azure.com)
2. Log in with your lab credentials
3. Search for **Public IP addresses**
4. Click **+ Create**
5. Fill in:
   * **Name:** `devops-pip`
   * **SKU:** Standard
   * **IP version:** IPv4
   * **Assignment:** Static
   * **Resource group:** (select your RG)
   * **Region:** (select correct region)
6. Click **Review + Create**
7. Click **Create**

***

### Best Practice Notes

* **Standard + Static** is recommended for production workloads
* Ensure the **region matches** the resources that will use this IP
* Public IPs incur cost—delete if unused

