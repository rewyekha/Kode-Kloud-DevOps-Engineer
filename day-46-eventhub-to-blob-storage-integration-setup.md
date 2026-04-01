# Day 46: EventHub to Blob Storage Integration Setup

## Azure: Event Hubs to Blob Storage Integration with VM Log Collection

> **Platform:** KodeKloud | **Cloud:** Microsoft Azure **Difficulty:** Intermediate | **Topic:** Azure Event Hubs, Blob Storage, Virtual Machines, Python, Log Ingestion, Backup

***

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
3. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
4. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
5. [Phase 1: Set Environment Variables](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-set-environment-variables)
6. [Phase 2: Create Event Hubs Namespace and Hub](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-create-event-hubs-namespace-and-hub)
7. [Phase 3: Set Up Azure Blob Storage](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-set-up-azure-blob-storage)
8. [Phase 4: Create the Virtual Machine](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-4-create-the-virtual-machine)
9. [Phase 5: Configure and Run the Log Script](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-5-configure-and-run-the-log-script)
10. [Phase 6: Verify Logs and Backup](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-6-verify-logs-and-backup)
11. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
12. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
13. [Resource Reference](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-reference)

***

### Lab Overview

The Nautilus DevOps team wants to integrate an Azure Virtual Machine with Azure Event Hubs and Azure Blob Storage for centralised log collection and backup. A Python script running on the VM sends logs simultaneously to an Event Hub for real-time ingestion and to a Blob Storage container for durable backup.

***

### Lab Objectives

1. Create an Event Hubs namespace named `datacenter-namespace` in East US with Standard tier and Auto-inflate enabled.
2. Create an Event Hub named `datacenter-hub` within the namespace.
3. Create a Storage Account named `datacenterst20647` in East US.
4. Create a Blob container named `datacenter-backup-9155` with public read access.
5. Create a Virtual Machine named `datacenter-vm` in East US.
6. Copy the `send_logs.py` script from the client host to the VM.
7. Modify the script with actual connection strings and run it multiple times.
8. Verify logs in Event Hubs metrics and confirm `logs.txt` in Blob Storage.

***

### Prerequisites

| Field              | Value                                                                |
| ------------------ | -------------------------------------------------------------------- |
| Portal URL         | `https://portal.azure.com/azurefreekmlprod.onmicrosoft.com`          |
| Username           | `kk_lab_user_main-436b7c0204604854@azurefreekmlprod.onmicrosoft.com` |
| Password           | `+AMNP4ZR`                                                           |
| Region             | `East US`                                                            |
| Resource Group     | `kml_rg_main-436b7c0204604854`                                       |
| Client script path | `/root/send_logs.py`                                                 |

***

### Architecture

```bash
┌──────────────────────────────────────────────────────────────────────┐
│                        Azure East US Region                          │
│                                                                      │
│   ┌─────────────────┐                                                │
│   │  Client Host    │                                                │
│   │  /root/         │                                                │
│   │  send_logs.py   │──── scp ────────────────────────────────┐      │
│   └─────────────────┘                                         │      │
│                                                               ▼      │
│   ┌───────────────────────────────────────────────────────────────┐  │
│   │               datacenter-vm (Ubuntu 22.04)                    │  │
│   │               Public IP: 20.120.103.248                       │  │
│   │               /home/azureuser/send_logs.py                    │  │
│   │                          │                                    │  │
│   │            ┌─────────────┴──────────────┐                     │  │
│   │            ▼                            ▼                     │  │
│   │   ┌─────────────────┐      ┌──────────────────────────┐      │  │
│   │   │ datacenter-     │      │   datacenterst20647       │     │  │
│   │   │ namespace       │      │   (Storage Account)       │     │  │
│   │   │ (Event Hubs)    │      │                           │     │  │
│   │   │                 │      │  datacenter-backup-9155   │     │  │
│   │   │ datacenter-hub  │      │  (Blob Container)         │     │  │
│   │   │ (Event Hub)     │      │  logs.txt                 │     │  │
│   │   └─────────────────┘      └──────────────────────────┘      │  │
│   └───────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

***

### Phase 1: Set Environment Variables

Define all resource names as shell variables upfront to avoid repetition across commands:

```bash
RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)
NAMESPACE="datacenter-namespace"
EVENT_HUB="datacenter-hub"
STORAGE_ACCOUNT="datacenterst20647"
CONTAINER="datacenter-backup-9155"
VM_NAME="datacenter-vm"

echo $RESOURCE_GROUP
```

**Terminal Output:**

```
~ ➜  RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)
NAMESPACE="datacenter-namespace"
EVENT_HUB="datacenter-hub"
STORAGE_ACCOUNT="datacenterst20647"
CONTAINER="datacenter-backup-9155"
VM_NAME="datacenter-vm"

echo $RESOURCE_GROUP
kml_rg_main-436b7c0204604854

~ ➜
```

The resource group `kml_rg_main-436b7c0204604854` was retrieved automatically from the subscription.

***

### Phase 2: Create Event Hubs Namespace and Hub

#### Step 1: Create Event Hubs Namespace

Create the namespace with Standard SKU, Auto-inflate enabled, and a maximum of 2 throughput units:

```bash
az eventhubs namespace create \
  --name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --location eastus \
  --sku Standard \
  --enable-auto-inflate true \
  --maximum-throughput-units 2
```

**Terminal Output:**

```bash
~ ➜  az eventhubs namespace create \
  --name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --location eastus \
  --sku Standard \
  --enable-auto-inflate true \
  --maximum-throughput-units 2
{
  "createdAt": "2026-01-06T19:20:14.4704452Z",
  "disableLocalAuth": false,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-436b7c0204604854/providers/Microsoft.EventHub/namespaces/datacenter-namespace",
  "isAutoInflateEnabled": true,
  "kafkaEnabled": true,
  "location": "eastus",
  "maximumThroughputUnits": 2,
  "name": "datacenter-namespace",
  "provisioningState": "Succeeded",
  "sku": {
    "capacity": 1,
    "name": "Standard",
    "tier": "Standard"
  },
  "status": "Active",
  "updatedAt": "2026-03-27T01:34:18Z",
  "zoneRedundant": true
}
```

Namespace `datacenter-namespace` created with `provisioningState: Succeeded`, Auto-inflate enabled, and zone redundancy active.

#### Step 2: Create Event Hub

```bash
az eventhubs eventhub create \
  --name $EVENT_HUB \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP
```

**Terminal Output:**

```bash
~ ➜  az eventhubs eventhub create \
  --name $EVENT_HUB \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP
{
  "createdAt": "2026-03-27T01:34:46.373Z",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-436b7c0204604854/providers/Microsoft.EventHub/namespaces/datacenter-namespace/eventhubs/datacenter-hub",
  "location": "eastus",
  "messageRetentionInDays": 7,
  "name": "datacenter-hub",
  "partitionCount": 4,
  "partitionIds": ["0", "1", "2", "3"],
  "status": "Active",
  "updatedAt": "2026-03-27T01:34:46.72Z"
}
```

Event Hub `datacenter-hub` created with 4 partitions and 7-day message retention.

#### Step 3: Get Event Hub Connection String

```bash
EH_CONN_STRING=$(az eventhubs namespace authorization-rule keys list \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE \
  --name RootManageSharedAccessKey \
  --query primaryConnectionString -o tsv)

echo $EH_CONN_STRING
```

**Terminal Output:**

```
~ ➜  EH_CONN_STRING=$(az eventhubs namespace authorization-rule keys list \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE \
  --name RootManageSharedAccessKey \
  --query primaryConnectionString -o tsv)

echo $EH_CONN_STRING
Endpoint=sb://datacenter-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=m8U7cUGE9Dw7CeVa0ixQUYP68e3mDjh+++AEhL7vi6M=
```

The connection string was retrieved and stored in `$EH_CONN_STRING` for use in the Python script.

***

### Phase 3: Set Up Azure Blob Storage

#### Step 4: Create Storage Account

```bash
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access true
```

**Terminal Output:**

```bash
~ ➜  az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access true
{
  "accessTier": "Hot",
  "allowBlobPublicAccess": true,
  "kind": "StorageV2",
  "location": "eastus",
  "name": "datacenterst20647",
  "primaryEndpoints": {
    "blob": "https://datacenterst20647.blob.core.windows.net/",
    "dfs": "https://datacenterst20647.dfs.core.windows.net/",
    "file": "https://datacenterst20647.file.core.windows.net/",
    "queue": "https://datacenterst20647.queue.core.windows.net/",
    "table": "https://datacenterst20647.table.core.windows.net/",
    "web": "https://datacenterst20647.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available"
}
```

Storage account `datacenterst20647` created with `Standard_LRS` SKU, `StorageV2` kind, public blob access enabled, and `provisioningState: Succeeded`.

#### Step 5: Get Storage Connection String

```bash
STORAGE_CONN_STRING=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query connectionString \
  -o tsv)

echo $STORAGE_CONN_STRING
```

**Terminal Output:**

```
~ ➜  STORAGE_CONN_STRING=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query connectionString \
  -o tsv)

echo $STORAGE_CONN_STRING
DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=datacenterst20647;AccountKey=MG+HNCoa0GTlbbRLoucAUp2lqGO9gbXgaP/5845EgLt4zHacRYoTiavcqdFijI2HPIv1ctpF+fXG+AStKtY9sg==;BlobEndpoint=https://datacenterst20647.blob.core.windows.net/;FileEndpoint=https://datacenterst20647.file.core.windows.net/;QueueEndpoint=https://datacenterst20647.queue.core.windows.net/;TableEndpoint=https://datacenterst20647.table.core.windows.net/
```

#### Step 6: Create Blob Container with Public Access

```bash
az storage container create \
  --name $CONTAINER \
  --connection-string "$STORAGE_CONN_STRING" \
  --public-access container
```

**Terminal Output:**

```
~ ➜  az storage container create \
  --name $CONTAINER \
  --connection-string "$STORAGE_CONN_STRING" \
  --public-access container
{
  "created": true
}
```

Container `datacenter-backup-9155` created with `public-access: container` — allowing anonymous read access to blobs within it.

***

### Phase 4: Create the Virtual Machine

#### Step 7: Create VM

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --os-disk-size-gb 30 \
  --storage-sku Standard_LRS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --location eastus
```

**Terminal Output:**

```bash
~ ➜  az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --os-disk-size-gb 30 \
  --storage-sku Standard_LRS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --location eastus
SSH key files '/root/.ssh/id_rsa' and '/root/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage, back up your keys to a safe location.
{
  "fqdns": "",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-436b7c0204604854/providers/Microsoft.Compute/virtualMachines/datacenter-vm",
  "location": "eastus",
  "macAddress": "7C-ED-8D-17-D0-68",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "20.120.103.248",
  "resourceGroup": "kml_rg_main-436b7c0204604854",
  "zones": ""
}
```

VM `datacenter-vm` created and running. Public IP: `20.120.103.248`, Private IP: `10.0.0.4`. SSH keys were auto-generated at `/root/.ssh/id_rsa`.

#### Step 8: Get VM Public IP

```bash
VM_PUBLIC_IP=$(az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --show-details \
  --query publicIps -o tsv)

echo $VM_PUBLIC_IP
```

**Terminal Output:**

```
~ ➜  VM_PUBLIC_IP=$(az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --show-details \
  --query publicIps -o tsv)

echo $VM_PUBLIC_IP
20.120.103.248
```

***

### Phase 5: Configure and Run the Log Script

#### Step 9: Inspect the Original Script on Client Host

```bash
cat /root/send_logs.py
```

**Terminal Output:**

```bash
~ ➜  cat /root/send_logs.py
import os
from azure.storage.blob import BlobServiceClient
from azure.eventhub import EventHubProducerClient, EventData

# Event Hub Configuration
eventhub_conn_str = "<Event Hub Connection String>"
eventhub_name = "datacenter-hub"
producer = EventHubProducerClient.from_connection_string(eventhub_conn_str, eventhub_name=eventhub_name)

# Blob Storage Configuration
blob_conn_str = "<Blob Storage Connection String>"
blob_service_client = BlobServiceClient.from_connection_string(blob_conn_str)
blob_client = blob_service_client.get_blob_client(container="datacenter-backup-9155", blob="logs.txt")

# Generate and Send Logs
log_data = "Log entry from VM\n"

# Send to Event Hub
event_data_batch = producer.create_batch()
event_data_batch.add(EventData(log_data))
producer.send_batch(event_data_batch)

# Backup to Blob Storage
blob_client.upload_blob(log_data, blob_type="AppendBlob", overwrite=True)

print("Log sent to Event Hub and backed up to Blob Storage.")
```

The script contains two placeholder values that need to be replaced with the actual connection strings before use.

#### Step 10: Copy Script to VM

```bash
scp -i ~/.ssh/id_rsa /root/send_logs.py azureuser@$VM_PUBLIC_IP:/home/azureuser/
```

**Terminal Output:**

```
~ ➜  scp -i ~/.ssh/id_rsa /root/send_logs.py azureuser@$VM_PUBLIC_IP:/home/azureuser/
The authenticity of host '20.120.103.248 (20.120.103.248)' can't be established.
ECDSA key fingerprint is SHA256:qiZHmsgq4ildsmgTstbnQC+WHuE26JG1dNaavKPkwu0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.120.103.248' (ECDSA) to the list of known hosts.
send_logs.py                                       100%  965     8.2KB/s   00:00
```

Script transferred at `8.2KB/s` — `100%` complete.

#### Step 11: SSH into the VM

```bash
ssh azureuser@$VM_PUBLIC_IP
```

**Terminal Output:**

```bash
~ ➜  ssh azureuser@$VM_PUBLIC_IP
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)

  System load:  0.55              Processes:             109
  Usage of /:   5.5% of 28.89GB  Users logged in:       0
  Memory usage: 31%               IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

azureuser@datacenter-vm:~$
```

#### Step 12: Update the Script with Connection Strings

Open the script and replace both placeholder values with the actual connection strings:

```bash
nano /home/azureuser/send_logs.py
```

The script after modification:

```python
import os
from azure.storage.blob import BlobServiceClient
from azure.eventhub import EventHubProducerClient, EventData

# Event Hub Configuration
eventhub_conn_str = "Endpoint=sb://datacenter-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=m8U7cUGE9Dw7CeVa0ixQUYP68e3mDjh+++AEhL7vi6M="
eventhub_name = "datacenter-hub"
producer = EventHubProducerClient.from_connection_string(eventhub_conn_str, eventhub_name=eventhub_name)

# Blob Storage Configuration
blob_conn_str = "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=datacenterst20647;AccountKey=MG+HNCoa0GTlbbRLoucAUp2lqGO9gbXgaP/5845EgLt4zHacRYoTiavcqdFijI2HPIv1ctpF+fXG+AStKtY9sg==;BlobEndpoint=https://datacenterst20647.blob.core.windows.net/;FileEndpoint=https://datacenterst20647.file.core.windows.net/;QueueEndpoint=https://datacenterst20647.queue.core.windows.net/;TableEndpoint=https://datacenterst20647.table.core.windows.net/"
blob_service_client = BlobServiceClient.from_connection_string(blob_conn_str)
blob_client = blob_service_client.get_blob_client(container="datacenter-backup-9155", blob="logs.txt")

# Generate and Send Logs
log_data = "Log entry from VM\n"

# Send to Event Hub
event_data_batch = producer.create_batch()
event_data_batch.add(EventData(log_data))
producer.send_batch(event_data_batch)

# Backup to Blob Storage
blob_client.upload_blob(log_data, blob_type="AppendBlob", overwrite=True)

print("Log sent to Event Hub and backed up to Blob Storage.")
```

#### Step 13: Install Python Dependencies

```bash
sudo apt install -y python3-pip
```

During installation, a system dialog appeared asking which services should be restarted after library updates:

```bash
Daemons using outdated libraries
Which services should be restarted?

[ ] networkd-dispatcher.service
[ ] unattended-upgrades.service
[*] walinuxagent.service
```

`Enter` was pressed to confirm `Ok` and allow `walinuxagent.service` to restart. Installation continued and completed successfully — `python3-pip 22.0.2` installed along with `build-essential` and related packages.

Install the Azure SDK libraries:

```bash
pip3 install azure-storage-blob azure-eventhub
```

**Terminal Output:**

```bash
azureuser@datacenter-vm:~$ pip3 install azure-storage-blob azure-eventhub
Defaulting to user installation because normal site-packages is not writeable
Collecting azure-storage-blob
  Downloading azure_storage_blob-12.28.0-py3-none-any.whl (431 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 431.5/431.5 KB 5.7 MB/s eta 0:00:00
Collecting azure-eventhub
  Downloading azure_eventhub-5.15.1-py3-none-any.whl (317 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 317.1/317.1 KB 14.4 MB/s eta 0:00:00
Collecting azure-core>=1.30.0
  Downloading azure_core-1.39.0-py3-none-any.whl (218 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 218.3/218.3 KB 14.4 MB/s eta 0:00:00
Collecting typing-extensions>=4.6.0
  Downloading typing_extensions-4.15.0-py3-none-any.whl (44 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 44.6/44.6 KB 2.8 MB/s eta 0:00:00
Collecting isodate>=0.6.1
  Downloading isodate-0.7.2-py3-none-any.whl (22 kB)
Requirement already satisfied: requests>=2.21.0 in /usr/lib/python3/dist-packages (from azure-core>=1.30.0->azure-storage-blob) (2.25.1)
Installing collected packages: typing-extensions, isodate, azure-core, azure-storage-blob, azure-eventhub
Successfully installed azure-core-1.39.0 azure-eventhub-5.15.1 azure-storage-blob-12.28.0 isodate-0.7.2 typing-extensions-4.15.0
```

All packages installed successfully: `azure-storage-blob 12.28.0`, `azure-eventhub 5.15.1`, `azure-core 1.39.0`.

#### Step 14: Run the Script Multiple Times

```bash
python3 /home/azureuser/send_logs.py
python3 /home/azureuser/send_logs.py
python3 /home/azureuser/send_logs.py
python3 /home/azureuser/send_logs.py
python3 /home/azureuser/send_logs.py
python3 /home/azureuser/send_logs.py
python3 /home/azureuser/send_logs.py
```

**Terminal Output:**

```bash
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log sent to Event Hub and backed up to Blob Storage.
azureuser@datacenter-vm:~$
```

The script ran 7 times, each printing `Log sent to Event Hub and backed up to Blob Storage.` — confirming successful delivery to both destinations on every execution.

***

### Phase 6: Verify Logs and Backup

#### Event Hubs Verification

In the Azure portal, navigate to: `Event Hubs` → `datacenter-namespace` → `datacenter-hub` → **Overview**

The metrics graphs confirm incoming requests and successful message delivery from the VM.

#### Blob Storage Verification

In the Azure portal, navigate to: `Storage accounts` → `datacenterst20647` → **Containers** → `datacenter-backup-9155`

The file `logs.txt` is present in the container, confirming log backup was successful.

***

### Lab Complete

| Requirement                | Resource                                | Configuration                          | Status    |
| -------------------------- | --------------------------------------- | -------------------------------------- | --------- |
| Event Hubs Namespace       | `datacenter-namespace`                  | Standard / Auto-inflate / East US      | Confirmed |
| Event Hub                  | `datacenter-hub`                        | 4 partitions / 7-day retention         | Confirmed |
| Storage Account            | `datacenterst20647`                     | Standard\_LRS / StorageV2 / East US    | Confirmed |
| Blob Container             | `datacenter-backup-9155`                | Public read access                     | Confirmed |
| Virtual Machine            | `datacenter-vm`                         | Ubuntu 22.04 / Standard\_B1s / East US | Confirmed |
| Script copied              | `/home/azureuser/send_logs.py`          | From `/root/send_logs.py` on client    | Confirmed |
| Connection strings updated | Both placeholders replaced              | EH + Blob conn strings                 | Confirmed |
| SDK installed              | `azure-eventhub` + `azure-storage-blob` | v5.15.1 + v12.28.0                     | Confirmed |
| Script executed            | 7 runs                                  | All returned success message           | Confirmed |
| Event Hub metrics          | Incoming messages                       | Spike visible in portal                | Confirmed |
| Blob Storage               | `logs.txt` in container                 | File present and accessible            | Confirmed |

***

### Key Concepts

#### Dual Destination Log Architecture

This lab demonstrates a common production pattern where logs are sent to two destinations simultaneously:

```bash
VM (send_logs.py)
      │
      ├──── Event Hub ────▶ Real-time stream processing
      │                     (Azure Stream Analytics, consumers)
      │
      └──── Blob Storage ──▶ Long-term durable backup
                             (queryable, archivable, cost-effective)
```

Event Hubs handles the real-time ingestion path while Blob Storage handles the durable archive path. Both are written in a single script execution.

#### AppendBlob vs BlockBlob

The script used `blob_type="AppendBlob"` when writing to Blob Storage:

| Blob Type    | Behaviour                       | Use Case                 |
| ------------ | ------------------------------- | ------------------------ |
| `BlockBlob`  | Replaces entire blob on upload  | Files, images, documents |
| `AppendBlob` | Appends new data to end of blob | Log files, audit trails  |
| `PageBlob`   | Random read/write access        | Azure VM disk files      |

`AppendBlob` is the correct choice for log files because multiple script executions append to the same `logs.txt` file rather than replacing it each time.

#### Event Hub Partitions

`datacenter-hub` was created with 4 partitions. Partitions are the unit of parallelism in Event Hubs:

```bash
datacenter-hub
├── Partition 0
├── Partition 1
├── Partition 2
└── Partition 3
```

Each partition is an ordered sequence of events. Multiple consumers can read from different partitions in parallel, enabling high-throughput processing. The Python SDK distributes events across partitions automatically.

#### Connection String Security

Both connection strings used in this lab follow the Azure Shared Access Signature (SAS) pattern:

```bash
Event Hub:
Endpoint=sb://<namespace>.servicebus.windows.net/
SharedAccessKeyName=RootManageSharedAccessKey
SharedAccessKey=<base64-encoded-key>

Blob Storage:
DefaultEndpointsProtocol=https
AccountName=<storage-account>
AccountKey=<base64-encoded-key>
BlobEndpoint=https://<account>.blob.core.windows.net/
```

In production, connection strings should be stored in **Azure Key Vault** rather than hardcoded in scripts, and Managed Identities should be used to authenticate the VM to Azure services without credentials.

#### `walinuxagent.service` Restart Dialog

During `apt install`, Ubuntu displayed a dialog asking which services to restart after library updates. The `walinuxagent.service` was pre-selected — this is the **Windows Azure Linux Agent**, which manages VM provisioning and communication with the Azure fabric. Restarting it during package installation is safe and expected on Azure VMs.

***

### Resource Reference

| Resource             | Type               | Value                          |
| -------------------- | ------------------ | ------------------------------ |
| Resource Group       | Azure RG           | `kml_rg_main-436b7c0204604854` |
| Event Hubs Namespace | Microsoft.EventHub | `datacenter-namespace`         |
| Event Hub            | Event Hub Instance | `datacenter-hub`               |
| Partition Count      | Event Hub          | `4`                            |
| Message Retention    | Event Hub          | `7 days`                       |
| Storage Account      | Microsoft.Storage  | `datacenterst20647`            |
| Storage SKU          | Tier               | `Standard_LRS`                 |
| Blob Container       | Container          | `datacenter-backup-9155`       |
| Public Access        | Container          | `container` (anonymous read)   |
| Log File             | Blob               | `logs.txt` (AppendBlob)        |
| Virtual Machine      | Microsoft.Compute  | `datacenter-vm`                |
| VM Image             | OS                 | `Ubuntu 22.04.5 LTS`           |
| VM Size              | Compute            | `Standard_B1s`                 |
| VM Public IP         | Network            | `20.120.103.248`               |
| VM Private IP        | Network            | `10.0.0.4`                     |
| Script Path on VM    | File               | `/home/azureuser/send_logs.py` |
| azure-eventhub       | Python SDK         | `v5.15.1`                      |
| azure-storage-blob   | Python SDK         | `v12.28.0`                     |
| azure-core           | Python SDK         | `v1.39.0`                      |
| Script executions    | Total runs         | `7`                            |

***

_Lab completed on 2026-03-27 | Azure Region: East US | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

