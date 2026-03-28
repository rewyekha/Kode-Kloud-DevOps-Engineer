# Day 47: SQL Database Migration and Setup

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to Azure. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. As part of this migration, they are focusing on setting up and managing Azure SQL Databases, implementing backup processes, and ensuring data recovery. Below are the tasks they require you to perform: Task 1: Create an Azure SQL Database

1. Create a publicly accessible Azure SQL Database instance with the following details:
   * Database Name: `devops-sqldb`.
   * Server Name: `devops-server-15936`.
   * Location: `West US`
   * Backup Storage Redundancy: Locally-redundant backup storage.
   * Hardware Configuration: Basic (For less demanding workloads).
   * Admin Username: `devops-admin`.
   * Admin Password: Set an appropriate password.
   * Database Size: Set to 2 GiB.
   * Keep all other configurations as `default`.
2. Ensure the database is in the `Ready` state. Task 2: Create a Storage Account
3. Create a Storage Account named `devopsst28401`.
4. Configure a Blob Container named `devops-container-12276` within this storage account. Task 3: Backup the Azure SQL Database
5. Take a backup of the Azure SQL Database instance `devops-sqldb` and store it in the Blob Container:
   * Storage Account: `devopsst28401`.
   * Blob Container: `devops-container-12276`.
   * Backup File Name: `devops-db-backup`.
6. Ensure the backup is fully exported to the blob container. Task 4: Download the Backup
7. Download the backup file from the Blob Container to the `/opt` directory on the `azure-client` host.
8. Ensure the file is accessible and properly named based on its extension. Requirements for Completion

* Ensure the SQL Database is in the `Ready` state.
* Confirm the backup is stored in the specified Blob Container.
* Verify the backup file is successfully downloaded to the `/opt` directory on the client host.&#x20;



## Azure SQL Database: Create, Backup, and Download to Local Storage

> **Platform:** KodeKloud | **Cloud:** Microsoft Azure **Difficulty:** Intermediate | **Topic:** Azure SQL Database, Blob Storage, Database Backup, BACPAC Export, Azure CLI

***

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
3. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
4. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
5. [Phase 1: Set Environment Variables](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-set-environment-variables)
6. [Phase 2: Create Azure SQL Server and Database](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-create-azure-sql-server-and-database)
7. [Phase 3: Create Storage Account and Blob Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-create-storage-account-and-blob-container)
8. [Phase 4: Export Database Backup to Blob Storage](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-4-export-database-backup-to-blob-storage)
9. [Phase 5: Download Backup to Client Host](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-5-download-backup-to-client-host)
10. [Phase 6: Verify Backup in Blob Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-6-verify-backup-in-blob-container)
11. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
12. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
13. [Resource Reference](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-reference)

***

### Lab Overview

The Nautilus DevOps team is migrating infrastructure to Azure in incremental steps. This lab covers setting up an Azure SQL Database, configuring blob storage for backups, exporting a database backup as a BACPAC file, and downloading it to the local client host for verification.

***

### Lab Objectives

**Task 1: Create an Azure SQL Database**

* Database name: `devops-sqldb`
* Server name: `devops-server-15936`
* Location: `West US`
* Backup storage redundancy: Locally-redundant
* Hardware configuration: Basic
* Admin username: `devops-admin`
* Database size: `2GB`
* Ensure database is in `Ready` state

**Task 2: Create a Storage Account**

* Storage account name: `devopsst28401`
* Blob container name: `devops-container-12276`

**Task 3: Backup the Azure SQL Database**

* Export `devops-sqldb` to Blob container `devops-container-12276`
* Backup file name: `devops-db-backup`

**Task 4: Download the Backup**

* Download backup to `/opt` directory on the `azure-client` host
* File must be accessible and properly named with its extension

***

### Prerequisites

| Field          | Value                                                                |
| -------------- | -------------------------------------------------------------------- |
| Portal URL     | `https://portal.azure.com`                                           |
| Username       | `kk_lab_user_main-bcdf442fc488436e@azurefreekmlprod.onmicrosoft.com` |
| Password       | `Z$M%NJC9`                                                           |
| Resource Group | `kml_rg_main-bcdf442fc488436e`                                       |
| Region         | `West US`                                                            |

***

### Architecture

```bash
┌─────────────────────────────────────────────────────────────────────┐
│                      Azure West US Region                           │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │          devops-server-15936 (Azure SQL Server)             │    │
│  │          devops-server-15936.database.windows.net           │    │
│  │                                                             │    │
│  │   ┌─────────────────────────────────────────────────────┐   │    │
│  │   │        devops-sqldb (Azure SQL Database)            │   │    │
│  │   │        Edition: Basic | Size: 2GB                   │   │    │
│  │   │        Status: Online                               │   │    │
│  │   └─────────────────────────────────────────────────────┘   │    │
│  └──────────────────────┬──────────────────────────────────────┘    │
│                         │ Export (BACPAC)                           │
│                         ▼                                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │          devopsst28401 (Storage Account)                    │    │
│  │                                                             │    │
│  │   ┌─────────────────────────────────────────────────────┐   │    │
│  │   │   devops-container-12276 (Blob Container)           │   │    │
│  │   │   devops-db-backup.bacpac (2770 bytes)              │   │    │
│  │   └─────────────────────────────────────────────────────┘   │    │
│  └──────────────────────┬──────────────────────────────────────┘    │
└─────────────────────────┼───────────────────────────────────────────┘
                          │ Download
                          ▼
              azure-client host
              /opt/devops-db-backup.bacpac
```

***

### Phase 1: Set Environment Variables

Define all resource names and configuration values as shell variables:

```bash
RESOURCE_GROUP="kml_rg_main-bcdf442fc488436e"
DATABASE="devops-sqldb"
SERVER_NAME="devops-server-15936"
ADMIN_USER="devops-admin"
ADMIN_PASS="Kodekloud@123"
STORAGE_ACCOUNT="devopsst28401"
CONTAINER="devops-container-12276"
LOCATION="westus"
DB_SIZE="2GB"
```

**Terminal Output:**

```bash
~ ➜  RESOURCE_GROUP="kml_rg_main-bcdf442fc488436e"
DATABASE="devops-sqldb"
SERVER_NAME="devops-server-15936"
ADMIN_USER="devops-admin"
ADMIN_PASS="Kodekloud@123"
STORAGE_ACCOUNT="devopsst28401"
CONTAINER="devops-container-12276"
LOCATION="westus"
DB_SIZE="2GB"

~ ➜
```

The resource group was first confirmed via `az group list` which returned `kml_rg_main-bcdf442fc488436e` in `eastus`.

***

### Phase 2: Create Azure SQL Server and Database

#### Step 1: Create the SQL Server

Create the logical SQL Server that will host the database. The server acts as a management endpoint and is separate from the database itself:

```bash
az sql server create \
  --name $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --admin-user $ADMIN_USER \
  --admin-password $ADMIN_PASS
```

**Terminal Output:**

```bash
~ ➜  az sql server create \
  --name $SERVER_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --admin-user $ADMIN_USER \
  --admin-password $ADMIN_PASS
{
  "administratorLogin": "devops-admin",
  "administratorLoginPassword": null,
  "externalGovernanceStatus": "Disabled",
  "fullyQualifiedDomainName": "devops-server-15936.database.windows.net",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-bcdf442fc488436e/providers/Microsoft.Sql/servers/devops-server-15936",
  "kind": "v12.0",
  "location": "westus",
  "minimalTlsVersion": "1.2",
  "name": "devops-server-15936",
  "publicNetworkAccess": "Enabled",
  "resourceGroup": "kml_rg_main-bcdf442fc488436e",
  "state": "Ready",
  "type": "Microsoft.Sql/servers",
  "version": "12.0"
}
```

SQL Server `devops-server-15936` created with FQDN `devops-server-15936.database.windows.net`, state `Ready`, and `publicNetworkAccess: Enabled`.

#### Step 2: Create a Firewall Rule

Allow all IP addresses to connect to the SQL Server. This ensures the export operation and client connections succeed:

```bash
az sql server firewall-rule create \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name AllowAll \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 255.255.255.255
```

**Terminal Output:**

```bash
~ ➜  az sql server firewall-rule create \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name AllowAll \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 255.255.255.255
{
  "endIpAddress": "255.255.255.255",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-bcdf442fc488436e/providers/Microsoft.Sql/servers/devops-server-15936/firewallRules/AllowAll",
  "name": "AllowAll",
  "resourceGroup": "kml_rg_main-bcdf442fc488436e",
  "startIpAddress": "0.0.0.0",
  "type": "Microsoft.Sql/servers/firewallRules"
}
```

Firewall rule `AllowAll` created covering the full IP range `0.0.0.0` to `255.255.255.255`.

#### Step 3: Create the SQL Database

Create the database on the server with Basic edition, 2GB size, and locally-redundant backup storage:

```bash
az sql db create \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name $DATABASE \
  --edition Basic \
  --max-size $DB_SIZE \
  --backup-storage-redundancy Local
```

**Terminal Output:**

```bash
~ ➜  az sql db create \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name $DATABASE \
  --edition Basic \
  --max-size $DB_SIZE \
  --backup-storage-redundancy Local
{
  "availabilityZone": "NoPreference",
  "catalogCollation": "SQL_Latin1_General_CP1_CI_AS",
  "collation": "SQL_Latin1_General_CP1_CI_AS",
  "creationDate": "2026-03-28T05:23:16.147000+00:00",
  "currentBackupStorageRedundancy": "Local",
  "currentServiceObjectiveName": "Basic",
  "currentSku": {
    "capacity": 5,
    "name": "Basic",
    "tier": "Basic"
  },
  "databaseId": "2865bfa9-f58a-4e19-9ffe-5070be85d616",
  "defaultSecondaryLocation": "eastus",
  "edition": "Basic",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-bcdf442fc488436e/providers/Microsoft.Sql/servers/devops-server-15936/databases/devops-sqldb",
  "location": "westus",
  "maxSizeBytes": 2147483648,
  "name": "devops-sqldb",
  "requestedBackupStorageRedundancy": "Local",
  "resourceGroup": "kml_rg_main-bcdf442fc488436e",
  "sku": {
    "capacity": 5,
    "name": "Basic",
    "tier": "Basic"
  },
  "status": "Online",
  "type": "Microsoft.Sql/servers/databases",
  "zoneRedundant": false
}
```

Database `devops-sqldb` created with `status: Online`, `edition: Basic`, `maxSizeBytes: 2147483648` (2GB), and `currentBackupStorageRedundancy: Local`.

#### Step 4: Verify Database Status

Confirm the database is in the required `Ready` state:

```bash
az sql db show \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name $DATABASE \
  --query status -o tsv
```

**Terminal Output:**

```
~ ➜  az sql db show \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name $DATABASE \
  --query status -o tsv
Online
```

The database status is `Online`, confirming it is ready for use.

***

### Phase 3: Create Storage Account and Blob Container

#### Step 5: Create the Storage Account

```bash
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
```

**Terminal Output:**

```bash
~ ➜  az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
{
  "accessTier": "Hot",
  "allowBlobPublicAccess": false,
  "creationTime": "2026-03-28T05:23:45.081869+00:00",
  "enableHttpsTrafficOnly": true,
  "kind": "StorageV2",
  "location": "westus",
  "name": "devopsst28401",
  "primaryEndpoints": {
    "blob": "https://devopsst28401.blob.core.windows.net/",
    "dfs": "https://devopsst28401.dfs.core.windows.net/",
    "file": "https://devopsst28401.file.core.windows.net/",
    "queue": "https://devopsst28401.queue.core.windows.net/",
    "table": "https://devopsst28401.table.core.windows.net/",
    "web": "https://devopsst28401.z22.web.core.windows.net/"
  },
  "primaryLocation": "westus",
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "type": "Microsoft.Storage/storageAccounts"
}
```

Storage account `devopsst28401` created with `provisioningState: Succeeded`, `Standard_LRS` SKU, and blob endpoint `https://devopsst28401.blob.core.windows.net/`.

#### Step 6: Get Storage Connection String

```bash
STORAGE_CONN=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query connectionString -o tsv)
```

**Terminal Output:**

```
~ ➜  STORAGE_CONN=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query connectionString -o tsv)

~ ➜
```

The connection string was stored in `$STORAGE_CONN` for use in subsequent container and blob operations.

#### Step 7: Create the Blob Container

```bash
az storage container create \
  --name $CONTAINER \
  --connection-string "$STORAGE_CONN"
```

**Terminal Output:**

```
~ ➜  az storage container create \
  --name $CONTAINER \
  --connection-string "$STORAGE_CONN"
{
  "created": true
}
```

Container `devops-container-12276` created successfully — `"created": true` confirms it did not already exist.

***

### Phase 4: Export Database Backup to Blob Storage

#### Step 8: Get Storage Account Key

Retrieve the storage account key required for the `az sql db export` command:

```bash
SA_KEY=$(az storage account keys list \
  --resource-group $RESOURCE_GROUP \
  --account-name $STORAGE_ACCOUNT \
  --query "[0].value" -o tsv)
```

**Terminal Output:**

```bash
~ ➜  SA_KEY=$(az storage account keys list \
  --resource-group $RESOURCE_GROUP \
  --account-name $STORAGE_ACCOUNT \
  --query "[0].value" -o tsv)

~ ➜
```

#### Step 9: Export the Database as BACPAC

Export the `devops-sqldb` database to the blob container as `devops-db-backup.bacpac`:

```bash
az sql db export \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name $DATABASE \
  --storage-key-type StorageAccessKey \
  --storage-key "$SA_KEY" \
  --storage-uri "https://$STORAGE_ACCOUNT.blob.core.windows.net/$CONTAINER/devops-db-backup.bacpac" \
  --admin-user $ADMIN_USER \
  --admin-password $ADMIN_PASS
```

**Terminal Output:**

```bash
~ ➜  az sql db export \
  --resource-group $RESOURCE_GROUP \
  --server $SERVER_NAME \
  --name $DATABASE \
  --storage-key-type StorageAccessKey \
  --storage-key "$SA_KEY" \
  --storage-uri "https://$STORAGE_ACCOUNT.blob.core.windows.net/$CONTAINER/devops-db-backup.bacpac" \
  --admin-user $ADMIN_USER \
  --admin-password $ADMIN_PASS
{
  "blobUri": "https://devopsst28401.blob.core.windows.net/devops-container-12276/devops-db-backup.bacpac",
  "databaseName": "devops-sqldb",
  "errorMessage": null,
  "id": "f4a6834d-0cca-4aa0-a9ef-34dbef22919b",
  "lastModifiedTime": "3/28/2026 5:30:02 AM",
  "name": "f4a6834d-0cca-4aa0-a9ef-34dbef22919b",
  "queuedTime": "3/28/2026 5:25:56 AM",
  "requestId": "f4a6834d-0cca-4aa0-a9ef-34dbef22919b",
  "requestType": "ExportDatabase",
  "serverName": "devops-server-15936",
  "status": "Completed",
  "type": "Microsoft.Sql/servers/databases/importExportOperationResults"
}
```

The export operation completed successfully:

* `status: Completed`
* `errorMessage: null`
* `blobUri: https://devopsst28401.blob.core.windows.net/devops-container-12276/devops-db-backup.bacpac`
* Queued at `5:25:56 AM`, completed at `5:30:02 AM` — approximately 4 minutes

***

### Phase 5: Download Backup to Client Host

#### Step 10: Download the BACPAC File

Download the backup file from the blob container to `/opt` on the azure-client host:

```bash
az storage blob download \
  --container-name $CONTAINER \
  --name devops-db-backup.bacpac \
  --file /opt/devops-db-backup.bacpac \
  --connection-string "$STORAGE_CONN"
```

**Terminal Output:**

```bash
~ ➜  az storage blob download \
  --container-name $CONTAINER \
  --name devops-db-backup.bacpac \
  --file /opt/devops-db-backup.bacpac \
  --connection-string "$STORAGE_CONN"
Finished[#############################################################]  100.0000%
{
  "container": "devops-container-12276",
  "content": "",
  "deleted": false,
  "name": "devops-db-backup.bacpac",
  "properties": {
    "blobType": "BlockBlob",
    "contentLength": 2770,
    "contentRange": "bytes None-None/2770",
    "contentSettings": {
      "contentMd5": "AOzQcT15vtLav83x9QY0tQ==",
      "contentType": "application/octet-stream"
    },
    "creationTime": "2026-03-28T05:29:45+00:00",
    "lastModified": "2026-03-28T05:29:45+00:00",
    "serverEncrypted": true
  }
}
```

Download completed at `100.0000%`. The file is `2770 bytes`, content type `application/octet-stream`, server encrypted, and stored as a `BlockBlob`.

#### Step 11: Confirm File on Client Host

```bash
ls -al /opt/devops-db-backup.bacpac
```

**Terminal Output:**

```
~ ➜  ls -al /opt/devops-db-backup.bacpac
-rw-r--r-- 1 root root 2770 Mar 28 05:31 /opt/devops-db-backup.bacpac
```

The file `devops-db-backup.bacpac` is present at `/opt/` with size `2770 bytes`, owned by root, created at `05:31`.

***

### Phase 6: Verify Backup in Blob Container

List all blobs in the container to confirm the backup file is present:

```bash
az storage blob list \
  --container-name $CONTAINER \
  --account-name $STORAGE_ACCOUNT \
  -o table
```

**Terminal Output:**

```bash
~ ➜  az storage blob list --container-name $CONTAINER --account-name $STORAGE_ACCOUNT -o table

There are no credentials provided in your command and environment, we will query for account key for your storage account.
It is recommended to provide --connection-string, --account-key or --sas-token in your command as credentials.

Name                     Blob Type    Blob Tier    Length    Content Type              Last Modified              Snapshot
-----------------------  -----------  -----------  --------  ------------------------  -------------------------  ----------
devops-db-backup.bacpac  BlockBlob    Hot          2770      application/octet-stream  2026-03-28T05:29:45+00:00
```

The blob `devops-db-backup.bacpac` is confirmed in the container:

* Type: `BlockBlob`
* Tier: `Hot`
* Size: `2770 bytes`
* Last modified: `2026-03-28T05:29:45`

> The credential warning is informational only — Azure CLI automatically queried the account key when no explicit credential flag was provided. The operation succeeded regardless.

***

### Lab Complete

| Task            | Requirement                                  | Result                          | Status    |
| --------------- | -------------------------------------------- | ------------------------------- | --------- |
| SQL Server      | `devops-server-15936` in West US             | Created, state `Ready`          | Confirmed |
| Firewall Rule   | `AllowAll` (0.0.0.0–255.255.255.255)         | Created                         | Confirmed |
| SQL Database    | `devops-sqldb`, Basic, 2GB, Local redundancy | Status `Online`                 | Confirmed |
| Storage Account | `devopsst28401`, Standard\_LRS, West US      | `provisioningState: Succeeded`  | Confirmed |
| Blob Container  | `devops-container-12276`                     | `"created": true`               | Confirmed |
| Database Export | `devops-db-backup.bacpac` to container       | `status: Completed`             | Confirmed |
| Backup in Blob  | `devops-db-backup.bacpac`                    | `2770 bytes`, `Hot` tier        | Confirmed |
| Download        | `/opt/devops-db-backup.bacpac`               | `2770 bytes`, `05:31` timestamp | Confirmed |

***

### Key Concepts

#### Azure SQL Database vs SQL Server (Logical)

In Azure, there is an important distinction between the SQL Server and the SQL Database:

| Resource                          | Description                                                                                |
| --------------------------------- | ------------------------------------------------------------------------------------------ |
| `Microsoft.Sql/servers`           | Logical container — manages firewall rules, admin credentials, and connectivity. Not a VM. |
| `Microsoft.Sql/servers/databases` | The actual database instance containing your data. Billed here.                            |

One logical SQL Server can host multiple databases. The server itself has no compute cost — only the databases within it are billed.

#### BACPAC vs BACPAC vs BAK

Azure SQL Database uses `BACPAC` as its portable export format:

| Format    | Tool               | Contains              | Use Case                       |
| --------- | ------------------ | --------------------- | ------------------------------ |
| `.bacpac` | `az sql db export` | Schema + data         | Migration, backup, portability |
| `.bak`    | SQL Server native  | Full/diff/log backups | On-premises SQL Server backup  |
| `.dacpac` | `sqlpackage`       | Schema only           | Deploy schema changes          |

`BACPAC` is the standard format for exporting Azure SQL Databases because it is portable across Azure SQL instances and on-premises SQL Server.

#### Backup Storage Redundancy Options

The `--backup-storage-redundancy Local` flag selected Locally Redundant Storage for backups:

| Option    | Copies   | Scope              | Use Case                     |
| --------- | -------- | ------------------ | ---------------------------- |
| `Local`   | 3 copies | Single datacenter  | Cost-sensitive, non-critical |
| `Zone`    | 3 copies | Availability zones | Higher availability          |
| `Geo`     | 6 copies | Two paired regions | Disaster recovery            |
| `GeoZone` | 6 copies | Zones + regions    | Maximum redundancy           |

For this lab, `Local` was appropriate as it minimises cost while still meeting the requirement.

#### The Export Workflow

```bash
devops-sqldb (Azure SQL Database)
        |
        | az sql db export
        | (uses StorageAccessKey for auth)
        |
        ▼
devops-container-12276/devops-db-backup.bacpac
(Blob Storage — Hot tier — BlockBlob)
        |
        | az storage blob download
        |
        ▼
/opt/devops-db-backup.bacpac
(azure-client host local filesystem)
```

The export is an asynchronous server-side operation. The CLI waits and polls until `status: Completed` is returned — in this lab it took approximately 4 minutes from queue to completion.

#### Why the Firewall Rule Was Required

Azure SQL Server blocks all inbound connections by default. The firewall rule `AllowAll` with IP range `0.0.0.0–255.255.255.255` was required to allow the export operation to proceed — the export service needs network access to the SQL Server to read and package the database contents into the BACPAC file.

***

### Resource Reference

| Resource            | Type                    | Value                                      |
| ------------------- | ----------------------- | ------------------------------------------ |
| Resource Group      | Azure RG                | `kml_rg_main-bcdf442fc488436e`             |
| SQL Server          | Microsoft.Sql/servers   | `devops-server-15936`                      |
| SQL Server FQDN     | DNS                     | `devops-server-15936.database.windows.net` |
| SQL Server Version  | Azure SQL               | `12.0`                                     |
| SQL Database        | Microsoft.Sql/databases | `devops-sqldb`                             |
| Database Edition    | SKU                     | `Basic`                                    |
| Database Size       | Max                     | `2GB (2147483648 bytes)`                   |
| Database Status     | Runtime                 | `Online`                                   |
| Backup Redundancy   | Storage                 | `Local`                                    |
| Database ID         | GUID                    | `2865bfa9-f58a-4e19-9ffe-5070be85d616`     |
| Firewall Rule       | AllowAll                | `0.0.0.0 – 255.255.255.255`                |
| Storage Account     | Microsoft.Storage       | `devopsst28401`                            |
| Storage SKU         | Tier                    | `Standard_LRS`                             |
| Blob Container      | Container               | `devops-container-12276`                   |
| Backup File         | Blob                    | `devops-db-backup.bacpac`                  |
| Backup Size         | Bytes                   | `2770`                                     |
| Backup Blob Type    | Azure Storage           | `BlockBlob`                                |
| Blob Tier           | Access                  | `Hot`                                      |
| Local Download Path | Filesystem              | `/opt/devops-db-backup.bacpac`             |
| Export Request ID   | GUID                    | `f4a6834d-0cca-4aa0-a9ef-34dbef22919b`     |
| Export Status       | Operation               | `Completed`                                |

***

_Lab completed on 2026-03-28 | Azure Region: West US | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>
