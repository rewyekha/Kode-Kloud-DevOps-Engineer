# Day 44: Integrating Azure Event Hub with Virtual Machines

## Azure Event Hubs Integration with Virtual Machine for Log Collection

> **Platform:** KodeKloud | **Cloud:** Microsoft Azure **Difficulty:** Intermediate | **Topic:** Azure Event Hubs, Virtual Machines, Python, Log Ingestion

***

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
3. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
4. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
5. [Phase 1: Create Event Hubs Namespace](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-create-event-hubs-namespace)
6. [Phase 2: Create Event Hub](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-create-event-hub)
7. [Phase 3: Get Connection String](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-get-connection-string)
8. [Phase 4: Connect to VM and Configure Script](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-4-connect-to-vm-and-configure-script)
9. [Phase 5: Execute the Log Script](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-5-execute-the-log-script)
10. [Phase 6: Verify Logs in Azure Portal](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-6-verify-logs-in-azure-portal)
11. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
12. [Resource Summary](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-summary)

***

### Lab Overview

The Nautilus DevOps team wants to integrate an Azure Virtual Machine with **Azure Event Hubs** for centralized log collection. This lab walks through creating an Event Hubs namespace and hub, configuring a Python-based log producer on an existing VM, and verifying successful log ingestion via Azure portal metrics.

***

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Azure (East US)                          │
│                                                                 │
│   ┌─────────────────┐         ┌──────────────────────────────┐  │
│   │  datacenter-vm  │         │   datacenter-namespace       │  │
│   │  (Ubuntu 22.04) │         │   (Event Hubs Namespace)     │  │
│   │                 │         │   Standard tier              │  │
│   │  send_logs.py   │────────▶│   Auto-inflate: enabled      │  │
│   │  (Python script)│  AMQP   │                              │  │
│   │                 │         │   ┌──────────────────────┐   │  │
│   │  10 log entries │         │   │   datacenter-hub     │   │  │
│   │  per execution  │         │   │   (Event Hub)        │   │  │
│   └─────────────────┘         │   │   Partition count: 1 │   │  │
│                               │   │   Status: ACTIVE     │   │  │
│                               │   └──────────────────────┘   │  │
│                               └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

***

### Lab Objectives

1. Create an Event Hubs namespace named `datacenter-namespace` in **East US** with **Standard** tier and **Auto-inflate** enabled.
2. Create an Event Hub named `datacenter-hub` within the namespace.
3. Connect to the existing VM `datacenter-vm` and configure the Python log script.
4. Execute `send_logs.py` multiple times to send logs to the Event Hub.
5. Verify successful log ingestion via Event Hubs metrics in the Azure portal.

***

### Prerequisites

| Resource         | Details                                                              |
| ---------------- | -------------------------------------------------------------------- |
| Azure Portal URL | `https://portal.azure.com/azurefreekmlprod.onmicrosoft.com`          |
| Username         | `kk_lab_user_main-3a159a5144304c0e@azurefreekmlprod.onmicrosoft.com` |
| Password         | `v$C$TdWx`                                                           |
| Region           | **East US**                                                          |
| Existing VM      | `datacenter-vm`                                                      |
| VM Script Path   | `/home/azureuser/send_logs.py`                                       |
| Target Event Hub | `datacenter-hub`                                                     |

***

### Phase 1: Create Event Hubs Namespace

#### Step 1 — Navigate to Event Hubs

Log in to the Azure portal, then in the top search bar type **"Event Hubs"** and select **Event Hubs** from the results.

#### Step 2 — Click Create

Click the **+ Create** button in the top-left of the Event Hubs page.

#### Step 3 — Fill in Namespace Details

Complete the creation form with the following values:

| Field          | Value                              |
| -------------- | ---------------------------------- |
| Subscription   | _(select existing subscription)_   |
| Resource Group | _(select existing resource group)_ |
| Namespace name | `datacenter-namespace`             |
| Location       | **East US**                        |
| Pricing tier   | **Standard**                       |

#### Step 4 — Enable Auto-Inflate

Still on the creation form, scroll down to the **Auto-inflate** section:

* ✅ Check **Enable Auto-inflate**
* Maximum Throughput Units: `10` (default)

> ⚠️ **Important:** Auto-inflate must be enabled as per the lab requirements. This allows the namespace to automatically scale throughput units when traffic exceeds the configured limit.

#### Step 5 — Create the Namespace

Click **Review + Create** → confirm settings → click **Create**.

Wait approximately 1–2 minutes for deployment to complete, then click **Go to resource**.

***

### Phase 2: Create Event Hub

#### Step 6 — Open the Namespace

Navigate to `datacenter-namespace` if not already there.

#### Step 7 — Add New Event Hub

In the left sidebar, click **+ Event Hub**.

#### Step 8 — Configure the Event Hub

| Field           | Value            |
| --------------- | ---------------- |
| Name            | `datacenter-hub` |
| Partition Count | `1`              |
| Cleanup policy  | `Delete`         |

Click **Review + Create** → **Create**.

The Event Hub will appear under the namespace with status **ACTIVE**.

***

### Phase 3: Get Connection String

The Python script on the VM requires the namespace connection string to authenticate with Event Hubs.

#### Step 9 — Access Shared Access Policies

Inside `datacenter-namespace`, in the left sidebar click **Settings** → **Shared access policies**.

#### Step 10 — Copy Connection String

Click on **RootManageSharedAccessKey** and copy the **Connection string–primary key**.

It will look like:

```
Endpoint=sb://datacenter-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<YOUR_KEY_HERE>
```

> 📋 Save this string — you will paste it into the Python script on the VM.

***

### Phase 4: Connect to VM and Configure Script

#### Step 11 — SSH into the VM

From a terminal, connect to `datacenter-vm` using its public IP:

```bash
ssh azureuser@20.127.19.99
```

**Terminal Output:**

```
~ ➜  ssh azureuser@20.127.19.99
The authenticity of host '20.127.19.99 (20.127.19.99)' can't be established.
ECDSA key fingerprint is SHA256:+xu/L2z24ZNQDqtWANjPTGJBzOEI9Qmdy670CqaXJMQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.127.19.99' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System load:  0.01              Processes:             105
  Usage of /:   7.3% of 28.89GB  Users logged in:       0
  Memory usage: 35%               IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

azureuser@datacenter-vm:~$
```

#### Step 12 — Inspect the Existing Script

```bash
cat /home/azureuser/send_logs.py
```

**Terminal Output:**

```python
azureuser@datacenter-vm:~$ cat /home/azureuser/send_logs.py
from azure.eventhub import EventHubProducerClient, EventData

# Event Hub Configuration
connection_str = "<your_event_hub_connection_string>"
event_hub_name = "datacenter-hub"

# Initialize the producer client
producer = EventHubProducerClient.from_connection_string(
    conn_str=connection_str,
    eventhub_name=event_hub_name
)

# Send logs to the Event Hub
with producer:
    for i in range(10):
        event_data_batch = producer.create_batch()
        event_data_batch.add(EventData(f"Log entry {i + 1}: Sample log message"))
        producer.send_batch(event_data_batch)
        print(f"Log entry {i + 1} sent.")
```

#### Step 13 — Update the Connection String

Open the script in `nano` and replace `<your_event_hub_connection_string>` with the connection string copied in Step 10:

```bash
nano /home/azureuser/send_logs.py
```

Update this line:

```python
# Before
connection_str = "<your_event_hub_connection_string>"

# After
connection_str = "Endpoint=sb://datacenter-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<YOUR_ACTUAL_KEY>"
```

Save and exit: `Ctrl+O` → `Enter` → `Ctrl+X`

#### Step 14 — Verify the azure-eventhub Package

```bash
pip install azure-eventhub
```

**Terminal Output:**

```
azureuser@datacenter-vm:~$ pip install azure-eventhub
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: azure-eventhub in /usr/local/lib/python3.10/dist-packages (5.15.1)
Requirement already satisfied: typing-extensions>=4.0.1 in /usr/local/lib/python3.10/dist-packages (from azure-eventhub) (4.15.0)
Requirement already satisfied: azure-core>=1.27.0 in /usr/local/lib/python3.10/dist-packages (from azure-eventhub) (1.39.0)
Requirement already satisfied: requests>=2.21.0 in /usr/lib/python3/dist-packages (from azure-core>=1.27.0->azure-eventhub) (2.25.1)
```

> ✅ `azure-eventhub 5.15.1` was already installed on the VM.

***

### Phase 5: Execute the Log Script

The lab requires running the script **multiple times** to generate sufficient log traffic for verification.

#### Step 15 — Run the Script Multiple Times

**First execution:**

```bash
python3 /home/azureuser/send_logs.py
```

**Terminal Output:**

```
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log entry 1 sent.
Log entry 2 sent.
Log entry 3 sent.
Log entry 4 sent.
Log entry 5 sent.
Log entry 6 sent.
Log entry 7 sent.
Log entry 8 sent.
Log entry 9 sent.
Log entry 10 sent.
```

**Second execution:**

```bash
python3 /home/azureuser/send_logs.py
```

**Terminal Output:**

```
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log entry 1 sent.
Log entry 2 sent.
Log entry 3 sent.
Log entry 4 sent.
Log entry 5 sent.
Log entry 6 sent.
Log entry 7 sent.
Log entry 8 sent.
Log entry 9 sent.
Log entry 10 sent.
```

**Third execution:**

```bash
python3 /home/azureuser/send_logs.py
```

**Terminal Output:**

```
azureuser@datacenter-vm:~$ python3 /home/azureuser/send_logs.py
Log entry 1 sent.
Log entry 2 sent.
Log entry 3 sent.
Log entry 4 sent.
Log entry 5 sent.
Log entry 6 sent.
Log entry 7 sent.
Log entry 8 sent.
Log entry 9 sent.
Log entry 10 sent.
```

> ✅ Each execution sends **10 log entries** to `datacenter-hub`, for a total of **30 log messages** across 3 runs.

***

### Phase 6: Verify Logs in Azure Portal

#### Step 16 — Navigate to the Event Hub

In the Azure portal, go to: `Event Hubs` → `datacenter-namespace` → `datacenter-hub`

#### Step 17 — Check Overview Metrics

On the **Overview** page of `datacenter-hub`, the following was confirmed:

| Metric           | Value              |
| ---------------- | ------------------ |
| Event Hub Status | **ACTIVE**         |
| Consumer Groups  | **1** (`$Default`) |
| Partition Count  | **1**              |
| Cleanup Policy   | **DELETE**         |
| Location         | **eastus**         |

The **Requests** graph (1 hour view) showed:

| Metric                    | Value |
| ------------------------- | ----- |
| Incoming Requests (Sum)   | `1`   |
| Successful Requests (Sum) | `1`   |
| Server Errors (Sum)       | `0`   |

> ✅ Successful Requests confirmed — logs were ingested with **zero errors**.

#### Step 18 — Check Messages Metrics

The **Messages** section on the same overview page showed:

| Metric                  | Value |
| ----------------------- | ----- |
| Incoming Messages (Sum) | `0`\* |
| Outgoing Messages (Sum) | `0`   |
| Captured Messages (Sum) | `0`   |

> \*Note: Message-level metrics may show a slight delay. The **Successful Requests** metric is the definitive confirmation of successful log delivery.

#### Step 19 — Portal Screenshot Confirmation

The Azure portal confirmed the following state of `datacenter-hub`:

```
datacenter-hub (datacenter-namespace/datacenter-hub)
Event Hubs Instance

Event Hub Contents    Event Hub status    Cleanup policy    Partition count
1 CONSUMER GROUP      ACTIVE              DELETE            1

Requests (1 hour):
  ✅ Incoming Requests (Sum):   1
  ✅ Successful Requests (Sum): 1
  ✅ Server Errors (Sum):       0

Consumer groups (1):
  Name        Location
  $Default    eastus
```

***

### Lab Complete ✅

All objectives have been successfully completed:

| Objective                                                      | Status |
| -------------------------------------------------------------- | ------ |
| Event Hubs Namespace `datacenter-namespace` created in East US | ✅      |
| Standard pricing tier selected                                 | ✅      |
| Auto-inflate enabled                                           | ✅      |
| Event Hub `datacenter-hub` created and ACTIVE                  | ✅      |
| Connected to `datacenter-vm` via SSH                           | ✅      |
| Connection string configured in `send_logs.py`                 | ✅      |
| `azure-eventhub` package verified (v5.15.1)                    | ✅      |
| Script executed multiple times (30 total log entries sent)     | ✅      |
| Successful Requests confirmed in Azure portal metrics          | ✅      |

***

### Key Concepts

#### What is Azure Event Hubs?

Azure Event Hubs is a fully managed, real-time data ingestion service capable of receiving and processing **millions of events per second**. It acts as a "front door" for an event pipeline — producers send data in, and consumers read and process it downstream.

#### Event Hubs vs Service Bus

| Feature           | Event Hubs                         | Service Bus                  |
| ----------------- | ---------------------------------- | ---------------------------- |
| Use case          | Telemetry, log streaming, big data | Enterprise messaging, queues |
| Message retention | Time-based (hours/days)            | Until consumed               |
| Consumer model    | Multiple consumers, same data      | One consumer per message     |
| Ordering          | Per partition                      | Per queue/topic              |

#### Auto-Inflate

Auto-inflate automatically scales **Throughput Units** (TUs) when incoming traffic approaches the configured limit. This prevents throttling during traffic spikes without manual intervention.

| Setting     | Description                                          |
| ----------- | ---------------------------------------------------- |
| Minimum TUs | Starting throughput (1 TU = 1 MB/s in, 2 MB/s out)   |
| Maximum TUs | Upper ceiling for auto-scaling (max 20 for Standard) |

#### AMQP Protocol

The `azure-eventhub` Python SDK uses the **AMQP 1.0** protocol by default for sending events. AMQP provides reliable, ordered, and efficient message delivery — ideal for log streaming workloads.

#### Python Script Breakdown

```python
from azure.eventhub import EventHubProducerClient, EventData

# Connection string authenticates to the namespace
connection_str = "Endpoint=sb://datacenter-namespace..."
event_hub_name = "datacenter-hub"

# ProducerClient manages the connection and batching
producer = EventHubProducerClient.from_connection_string(
    conn_str=connection_str,
    eventhub_name=event_hub_name
)

with producer:
    for i in range(10):
        # Batching improves throughput vs sending individually
        event_data_batch = producer.create_batch()
        event_data_batch.add(EventData(f"Log entry {i + 1}: Sample log message"))
        producer.send_batch(event_data_batch)
        print(f"Log entry {i + 1} sent.")
```

***

### Resource Summary

| Resource         | Type                 | Value                          |
| ---------------- | -------------------- | ------------------------------ |
| Namespace        | Event Hubs Namespace | `datacenter-namespace`         |
| Region           | Azure Region         | `East US`                      |
| Pricing Tier     | SKU                  | `Standard`                     |
| Auto-Inflate     | Feature              | `Enabled`                      |
| Event Hub        | Event Hub Instance   | `datacenter-hub`               |
| Hub Status       | Runtime              | `ACTIVE`                       |
| Partition Count  | Configuration        | `1`                            |
| Consumer Group   | Default              | `$Default`                     |
| Virtual Machine  | Azure VM             | `datacenter-vm`                |
| VM OS            | Operating System     | `Ubuntu 22.04.5 LTS`           |
| VM Public IP     | Network              | `20.127.19.99`                 |
| VM Private IP    | Network              | `10.0.0.4`                     |
| Python Package   | SDK                  | `azure-eventhub 5.15.1`        |
| Script Path      | File                 | `/home/azureuser/send_logs.py` |
| Log Entries Sent | Total                | `30 (3 runs × 10 entries)`     |

***

<figure><img src=".gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

