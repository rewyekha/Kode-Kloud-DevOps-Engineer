# Day 41: Working with Azure Table Storage

The Nautilus DevOps team is developing a simple 'To-Do' application using Azure Table Storage to store and manage tasks efficiently. The team needs to create an Azure Table to hold tasks, each identified by a unique `taskId`. Each task will have a description and a status, which indicates the progress of the task (e.g., 'completed' or 'in-progress').

Your task is to:

1. Create an Azure Storage Account named `nautilustablest8855` with a Table Storage table called `tasks`.
2. Insert the following tasks into the table:
   * Task 1: PartitionKey: 'tasks', RowKey: '1', description: 'Learn Table Storage', status: 'completed'
   * Task 2: PartitionKey: 'tasks', RowKey: '2', description: 'Build To-Do App', status: 'in-progress'
3. Verify that Task 1 has a status of 'completed' and Task 2 has a status of 'in-progress'.

**Note**: Use the Azure CLI to insert these tasks into the table.

Use the Azure Portal URL and login credentials below:

\
`Notes:`

* Create the resources only in the `eastus` region.



## Working with Azure Table Storage (To-Do Application)

### Objective

The objective of this lab is to create an Azure Storage Account, configure Azure Table Storage, and insert and verify task entities using Azure CLI.

The Nautilus DevOps team implemented a simple To-Do application using Azure Table Storage, where each task is stored as an entity with a unique RowKey.

***

### Prerequisites

* Azure CLI installed and configured
* Access to Azure subscription via portal login
* Resource Group already available in East US region

***

### Azure Portal Access Details

* Portal URL: [https://portal.azure.com/azurefreekmlprod.onmicrosoft.com](https://portal.azure.com/azurefreekmlprod.onmicrosoft.com)
* Username: [kml\_lab\_user\_main-68b943efc49943df@azurefreekmlprod.onmicrosoft.com](mailto:kml_lab_user_main-68b943efc49943df@azurefreekmlprod.onmicrosoft.com)
* Region requirement: East US

***

### Step 1: Verify Existing Resource Group

Command:

```bash
az group list
```

Output:

```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-68b943efc49943df",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-68b943efc49943df",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

Selected Resource Group:

```
kml_rg_main-68b943efc49943df
```

***

### Step 2: Define Environment Variables

```bash
RG="kml_rg_main-68b943efc49943df"
LOCATION="eastus"
STORAGE_ACCOUNT="nautilustablest8855"
TABLE_NAME="tasks"
```

***

### Step 3: Create Storage Account

Command:

```bash
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RG \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
```

Output:

```json
{
  "accessTier": "Hot",
  "creationTime": "2026-03-21T17:06:44.714917+00:00",
  "enableHttpsTrafficOnly": true,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-68b943efc49943df/providers/Microsoft.Storage/storageAccounts/nautilustablest8855",
  "kind": "StorageV2",
  "location": "eastus",
  "name": "nautilustablest8855",
  "provisioningState": "Succeeded",
  "resourceGroup": "kml_rg_main-68b943efc49943df",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  }
}
```

***

### Step 4: Retrieve Storage Account Key

Command:

```bash
ACCOUNT_KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name $STORAGE_ACCOUNT \
  --query "[0].value" -o tsv)
```

No output is returned (key stored in variable).

***

### Step 5: Create Azure Table

Command:

```bash
az storage table create \
  --name $TABLE_NAME \
  --account-name $STORAGE_ACCOUNT \
  --account-key $ACCOUNT_KEY
```

Output:

```json
{
  "created": true
}
```

***

### Step 6: Insert Task 1

Command:

```bash
az storage entity insert \
  --account-name $STORAGE_ACCOUNT \
  --account-key $ACCOUNT_KEY \
  --table-name $TABLE_NAME \
  --entity PartitionKey=tasks RowKey=1 description="Learn Table Storage" status="completed"
```

Output:

```json
{
  "date": "2026-03-21T17:07:33+00:00",
  "etag": "W/\"datetime'2026-03-21T17%3A07%3A33.9787628Z'\"",
  "version": "2019-02-02"
}
```

***

### Step 7: Insert Task 2

Command:

```bash
az storage entity insert \
  --account-name $STORAGE_ACCOUNT \
  --account-key $ACCOUNT_KEY \
  --table-name $TABLE_NAME \
  --entity PartitionKey=tasks RowKey=2 description="Build To-Do App" status="in-progress"
```

Output:

```json
{
  "date": "2026-03-21T17:07:44+00:00",
  "etag": "W/\"datetime'2026-03-21T17%3A07%3A44.9925572Z'\"",
  "version": "2019-02-02"
}
```

***

### Step 8: Verify Table Entities

Command:

```bash
az storage entity query \
  --account-name $STORAGE_ACCOUNT \
  --account-key $ACCOUNT_KEY \
  --table-name $TABLE_NAME
```

Output:

```json
{
  "items": [
    {
      "PartitionKey": "tasks",
      "RowKey": "1",
      "Timestamp": "2026-03-21T17:07:33.978762+00:00",
      "description": "Learn Table Storage",
      "etag": "W/\"datetime'2026-03-21T17%3A07%3A33.9787628Z'\"",
      "status": "completed"
    },
    {
      "PartitionKey": "tasks",
      "RowKey": "2",
      "Timestamp": "2026-03-21T17:07:44.992557+00:00",
      "description": "Build To-Do App",
      "etag": "W/\"datetime'2026-03-21T17%3A07%3A44.9925572Z'\"",
      "status": "in-progress"
    }
  ],
  "nextMarker": {}
}
```

***

### Result

The Azure Storage Account and Table Storage setup was completed successfully. Two task entities were inserted and verified using Azure CLI. Both tasks are correctly stored with appropriate PartitionKey and RowKey values.

***

### Conclusion

This exercise demonstrated how to:

* Create an Azure Storage Account using Azure CLI
* Configure Azure Table Storage
* Insert structured NoSQL entities
* Query and validate stored data

The lab is completed successfully with all requirements met.

<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

