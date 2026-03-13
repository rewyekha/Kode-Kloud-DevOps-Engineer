# Day 30: Create Azure SQL Database

The Nautilus Devops team is strategizing the migration of a portion of their infrastructure to Azure. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. Recently, they started working on creating and configuring some database instances on Azure.

For this task, create one `publicly` accessible Azure SQL Database instance along with the following details:

1\) The name of the Azure SQL Database must be `xfusion-sqldb`.

2\) The server name must be `xfusion-server-30206` under `centralus`.

3\) The compute + storage configuration should be **Basic (For less demanding workloads)**.

4\) The backup storage redundancy should be **Locally-redundant backup storage**.

5\) Set the login admin username to `xfusion-admin` and set an appropriate password.

6\) Set the database size to **2 GiB**.

7\) Keep the rest of the configurations as `default`. Finally, make sure the database is in the `Ready` state before submitting this task.

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)



| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-45db2c29c0f34520@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-45db2c29c0f34520@azurefreekmlprod.onmicrosoft.com) |
| Password   | QRF984vm                                                                                                                                           |
| Start Time | Wed Mar 04 03:42:07 UTC 2026                                                                                                                       |
| End Time   | Wed Mar 04 04:42:07 UTC 2026                                                                                                                       |

***

## 📘 Azure SQL Database Deployment – xfusion-sqldb

### 📝 Scenario

The Nautilus DevOps team needs to create a publicly accessible Azure SQL Database instance for development purposes.

**Requirements:**

* **Database name:** `xfusion-sqldb`
* **Server name:** `xfusion-server-30206`
* **Region:** `centralus`
* **Tier:** Basic (for small workloads)
* **Backup storage:** Locally-redundant
* **Admin login:** `xfusion-admin`
* **Database size:** 2 GiB
* **Public access:** Enabled

***

## 1️⃣ GUI Method

#### Step 1: Login to Azure Portal

* URL: [https://portal.azure.com](https://portal.azure.com)
* Credentials: Provided `kk_lab_user_main-45db2c29c0f34520@azurefreekmlprod.onmicrosoft.com`
* Explanation: The portal is the main interface to manage Azure resources.

***

#### Step 2: Create a SQL Server

1. Navigate to **Create a resource → Databases → SQL Server**.
2. Fill in details:
   * **Server name:** `xfusion-server-30206`
   * **Admin login:** `xfusion-admin`
   * **Password:** Set securely
   * **Location:** `centralus`

**Explanation:** The server acts as a container for databases. Admin credentials are required to access the databases.

***

#### Step 3: Create a SQL Database

1. Navigate to **Create a resource → Databases → SQL Database**.
2. Fill in details:
   * **Database name:** `xfusion-sqldb`
   * **Server:** Select `xfusion-server-30206`
   * **Compute + storage:** Basic
   * **Max size:** 2 GiB
   * **Backup storage:** Locally-redundant

**Explanation:** The Basic tier is suitable for development or low-demand workloads. Locally-redundant backup keeps a copy within the same region.

***

#### Step 4: Configure Networking / Public Access

1. Enable **Allow Azure services and resources to access this server**.
2. Optionally add client IP for firewall rule.

**Explanation:** Public access is required if you want external tools to connect to the database.

***

#### Step 5: Deployment & Status Check

* Click **Review + Create → Create**.
* Monitor deployment status from the portal. Wait until the database status is **Ready**.

**Explanation:** Deployment can take a few minutes. The status will show “Creating” during deployment and “Online” once ready.

***

## 2️⃣ CLI Verification Method

Once the database is deployed via GUI, you can verify it using **Azure CLI**.

***

#### Step 1: Login and Check Account

```bash
az login
az account show --output table
```

**Explanation:** Ensures you are connected to the correct subscription.

***

#### Step 2: Verify Database Exists

```bash
RESOURCE_GROUP="kml_rg_main-45db2c29c0f34520"
SERVER_NAME="xfusion-server-30206"
DB_NAME="xfusion-sqldb"

az sql db list \
  --server $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --output table
```

**Explanation:** Lists all databases under the server.

***

#### Step 3: Show Detailed Database Info

```bash
az sql db show \
  --name $DB_NAME \
  --server $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --output json | jq '{
    Name: .name,
    Status: .status,
    Location: .location,
    MaxSizeBytes: .maxSizeBytes,
    Edition: .edition,
    CurrentSku: .currentSku.name,
    BackupStorage: .currentBackupStorageRedundancy
}'
```

**Explanation:** Outputs key properties including status, size, tier, and backup type.

***

#### Step 4: Verify Backup Storage

```bash
az sql db show \
  --name $DB_NAME \
  --server $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "currentBackupStorageRedundancy" \
  --output table
```

**Explanation:** Confirms that backups are **LocallyRedundant**.

***

#### Step 5: Verify Server Info

```bash
az sql server show \
  --name $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --output table
```

**Explanation:** Confirms server location and basic info.

***

#### Step 6: Verify Firewall / Public Access

```bash
az sql server firewall-rule list \
  --server $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --output table
```

**Explanation:** Ensures the database is publicly accessible from required IPs.

***

#### Step 7: Watch Database Status (Optional)

```bash
watch -n 5 "az sql db show \
  --name $DB_NAME \
  --server $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --query status \
  --output table"
```

**Explanation:** Continuously checks database status until it shows **Online**.

***

## ✅ Verification Checklist

| Requirement     | Expected Result          |
| --------------- | ------------------------ |
| Database exists | xfusion-sqldb            |
| Status          | Online                   |
| Region          | centralus                |
| Tier            | Basic                    |
| Max Size        | 2 GiB (2147483648 bytes) |
| Backup          | LocallyRedundant         |
| Server          | xfusion-server-30206     |
| Public Access   | Firewall rule exists     |

***

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

**REVIEW:**

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>





<figure><img src=".gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>
