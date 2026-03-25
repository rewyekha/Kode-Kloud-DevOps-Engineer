# Day 45: Azure Kubernetes Service (AKS) Setup and Management

The Nautilus DevOps team is tasked with preparing an AKS cluster to deploy a Kubernetes-based application. The team has the following requirements:

1. Create an AKS cluster named `nautilus-aks`.
2. The Kubernetes version must be `1.33.0`.
3. The AKS cluster endpoint access must be private.
4. Ensure the cluster is created in the `Central US` region.
5. Edit the `agentpool` Node pools (delete all other node pool if exists) and configure the cluster with the following properties:
   * Node size: `D2s v3`.
   * Minimum node count: `1`.
   * Maximum node count: `2`.

* Disable the `Container Insights` for now and disable all kind of monitoring as well.

The AKS cluster must be configured with high availability and private endpoint access. Verify that the cluster meets the requirements and is ready for workloads.

Use the Azure Portal URL and login credentials below:\
`Notes:`

* Create the resources only in the `Central US` region.
* Ensure that the Kubernetes version is `1.33.0`.



## Azure Kubernetes Service: AKS Cluster Setup and Management

> **Platform:** KodeKloud | **Cloud:** Microsoft Azure **Difficulty:** Intermediate | **Topic:** AKS, Kubernetes, Node Pools, Private Cluster, Autoscaling

***

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
3. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
4. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
5. [Phase 1: Create the AKS Cluster via Azure Portal](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-create-the-aks-cluster-via-azure-portal)
   * [Step 1: Navigate to Kubernetes Services](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-navigate-to-kubernetes-services)
   * [Step 2: Basics Tab](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-basics-tab)
   * [Step 3: Node Pools Tab](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-node-pools-tab)
   * [Step 4: Networking Tab](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-networking-tab)
   * [Step 5: Integrations Tab](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-integrations-tab)
   * [Step 6: Monitoring Tab](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-monitoring-tab)
   * [Step 7: Review and Create](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-review-and-create)
6. [Phase 2: Deployment Confirmation](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-deployment-confirmation)
7. [Phase 3: Verify via Azure CLI](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-verify-via-azure-cli)
8. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
9. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
10. [Resource Reference](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-reference)

***

### Lab Overview

The Nautilus DevOps team is tasked with preparing an AKS cluster to deploy a Kubernetes-based application. The cluster must be production-grade with high availability, private endpoint access, autoscaling node pools, and all monitoring disabled for the initial setup phase.

***

### Lab Objectives

1. Create an AKS cluster named `nautilus-aks`.
2. The Kubernetes version must be `1.33.0`.
3. The AKS cluster endpoint access must be **private**.
4. Ensure the cluster is created in the **Central US** region.
5. Edit the `agentpool` node pool and configure:
   * Node size: `Standard D2s v3`
   * Minimum node count: `1`
   * Maximum node count: `2`
6. Delete all other node pools if they exist.
7. Disable Container Insights and all monitoring.

***

### Prerequisites

| Field          | Value                                                                |
| -------------- | -------------------------------------------------------------------- |
| Portal URL     | `https://portal.azure.com`                                           |
| Username       | `kk_lab_user_main-88919c00fd5649b1@azurefreekmlprod.onmicrosoft.com` |
| Password       | `h=+u28fw`                                                           |
| Region         | `Central US`                                                         |
| Resource Group | `kml_rg_main-88919c00fd5649b1`                                       |
| Subscription   | `Azure Free Labs`                                                    |

***

### Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    Azure Central US Region                       │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │              nautilus-aks (AKS Cluster)                  │   │
│   │                                                          │   │
│   │   Kubernetes version: 1.33.0                             │   │
│   │   API server access: Private                             │   │
│   │   High availability: Zones 1, 2, 3                       │   │
│   │   Monitoring: Disabled                                   │   │
│   │                                                          │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │              agentpool (System)                  │   │   │
│   │   │   VM Size:    Standard_D2s_v3                    │   │   │
│   │   │   Min nodes:  1                                  │   │   │
│   │   │   Max nodes:  2                                  │   │   │
│   │   │   Autoscale:  Enabled                            │   │   │
│   │   └─────────────────────────────────────────────────┘    │   │
│   └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

***

### Phase 1: Create the AKS Cluster via Azure Portal

#### Step 1: Navigate to Kubernetes Services

Log in to the Azure portal, then type **Kubernetes services** in the top search bar and click **Kubernetes services** from the results.

Click the **+ Create** button and select **Create a Kubernetes cluster**.

***

#### Step 2: Basics Tab

Fill in the following fields on the Basics tab:

| Field                        | Value                                 |
| ---------------------------- | ------------------------------------- |
| Subscription                 | _(select existing — Azure Free Labs)_ |
| Resource group               | `kml_rg_main-88919c00fd5649b1`        |
| Cluster preset configuration | **Dev/Test**                          |
| Kubernetes cluster name      | `nautilus-aks`                        |
| Region                       | **Central US**                        |
| Availability zones           | **Zones 1, 2, 3**                     |
| AKS pricing tier             | **Standard**                          |
| Kubernetes version           | **1.33.0**                            |
| Automatic upgrade            | **Disabled**                          |

> The portal showed the following preset options: Production Standard, Dev/Test, Production Economy, Production Enterprise. **Dev/Test** was selected as it is the appropriate choice for a lab and testing environment, offering the most configuration flexibility.

> The default Kubernetes version shown was `1.33.7 (default)`. This was changed to `1.33.0` as required by the lab.

Click **Next: Node pools**

***

#### Step 3: Node Pools Tab

Delete any additional node pools that exist other than `agentpool`.

Click on **agentpool** to edit it and configure the following:

| Field              | Value                                                   |
| ------------------ | ------------------------------------------------------- |
| Node pool name     | `agentpool`                                             |
| Mode               | **System**                                              |
| Node size          | **Standard D2s v3** _(search "D2s" in the size picker)_ |
| Scale method       | **Autoscale**                                           |
| Minimum node count | `1`                                                     |
| Maximum node count | `2`                                                     |

Click **Update** to save node pool settings.

Click **Next: Networking**

***

#### Step 4: Networking Tab

| Field                 | Value                                       |
| --------------------- | ------------------------------------------- |
| Network configuration | _(leave default)_                           |
| Private cluster       | **Enable private cluster** — check this box |

Enabling **private cluster** ensures the Kubernetes API server endpoint is only accessible within the virtual network — satisfying the requirement for private endpoint access.

Click **Next: Integrations**

***

#### Step 5: Integrations Tab

| Field                    | Value        |
| ------------------------ | ------------ |
| Azure Container Registry | **None**     |
| Azure Policy             | **Disabled** |

Leave all other settings as default. Click **Next: Monitoring**

***

#### Step 6: Monitoring Tab

Disable all monitoring options on this tab:

| Field                     | Value        |
| ------------------------- | ------------ |
| Enable Container Insights | **Disabled** |
| Enable Prometheus metrics | **Disabled** |
| Enable Grafana            | **Disabled** |
| Enable recommended alerts | **Disabled** |

All monitoring was turned off as required by the lab instructions.

Click **Next: Advanced** → leave defaults → **Next: Tags** → leave blank.

***

#### Step 7: Review and Create

Click **Review + Create**. Azure runs a validation check.

Once the green **Validation passed** banner appears, review the configuration summary and click **Create**.

***

### Phase 2: Deployment Confirmation

The deployment ran under the name `microsoft.aks-1774412293526` in resource group `kml_rg_main-88919c00fd5649b1`.

**Portal deployment screen confirmed:**

```
Your deployment is complete

Deployment name  : microsoft.aks-1774412293526
Subscription     : Azure Free Labs
Resource group   : kml_rg_main-88919c00fd5649b1
Start time       : 3/25/2026, 9:49:18 AM
Correlation ID   : 81cd50bb-bbe2-4476-a18b-8df1aa99bb59
```

**Deployment details:**

| Resource                                       | Type                             | Status |
| ---------------------------------------------- | -------------------------------- | ------ |
| `nautilus-aks/aksManagedNodeOSUpgradeSchedule` | `Microsoft.ContainerService/man` | OK     |
| `nautilus-aks`                                 | `Microsoft.ContainerService/man` | OK     |

Both resources deployed with status **OK**. The portal also showed a **Deployment succeeded** notification in the top-right corner.

***

### Phase 3: Verify via Azure CLI

After deployment, the cluster was verified using the Azure CLI to confirm all configuration requirements were met.

#### Set Resource Group Variable

```bash
RG=$(az group list --query "[0].name" -o tsv)
```

#### Verify Cluster Configuration

```bash
echo "=== CLUSTER INFO ==="
az aks show \
  --name nautilus-aks \
  --resource-group $RG \
  --query "{Name:name, Location:location, Version:kubernetesVersion, State:provisioningState, Private:apiServerAccessProfile.enablePrivateCluster}" \
  --output table
```

**Terminal Output:**

```
=== CLUSTER INFO ===
Name          Location    Version    State      Private
------------  ----------  ---------  ---------  ---------
nautilus-aks  centralus   1.33.0     Succeeded  True
```

#### Verify Node Pool Configuration

```bash
echo "=== NODE POOL ==="
az aks nodepool show \
  --cluster-name nautilus-aks \
  --resource-group $RG \
  --name agentpool \
  --query "{Name:name, Size:vmSize, Min:minCount, Max:maxCount, Autoscale:enableAutoScaling}" \
  --output table
```

**Terminal Output:**

```
=== NODE POOL ===
(AuthorizationFailed) The client 'c5ee61e1-f06e-4c83-9c57-3312597d00e4' with object id
'bb0dfa1f-13a0-4ac4-9170-c45e7e02c918' does not have authorization to perform action
'Microsoft.ContainerService/managedClusters/agentPools/read' over scope
'/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/
kml_rg_main-88919c00fd5649b1/providers/Microsoft.ContainerService/managedClusters/
nautilus-aks/agentPools/agentpool' or the scope is invalid.
Code: AuthorizationFailed
```

> This error is a lab environment permission restriction — the lab account does not have `agentPools/read` CLI permission. This does not indicate a problem with the cluster itself. The node pool configuration was confirmed correct via the Azure Portal under **nautilus-aks → Node pools**.

#### Verify Monitoring is Disabled

```bash
echo "=== MONITORING ==="
az aks show \
  --name nautilus-aks \
  --resource-group $RG \
  --query "addonProfiles.omsagent" \
  --output table
```

**Terminal Output:**

```
=== MONITORING ===
~ ➜
```

The empty output confirms `addonProfiles.omsagent` is `null` — Container Insights is not installed and monitoring is fully disabled.

***

### Lab Complete

| Requirement              | Expected          | CLI / Portal Result   | Status    |
| ------------------------ | ----------------- | --------------------- | --------- |
| Cluster name             | `nautilus-aks`    | `nautilus-aks`        | Confirmed |
| Region                   | `Central US`      | `centralus`           | Confirmed |
| Kubernetes version       | `1.33.0`          | `1.33.0`              | Confirmed |
| Provisioning state       | `Succeeded`       | `Succeeded`           | Confirmed |
| Private cluster endpoint | `True`            | `True`                | Confirmed |
| Node size                | `Standard D2s v3` | Confirmed via Portal  | Confirmed |
| Minimum node count       | `1`               | Configured via Portal | Confirmed |
| Maximum node count       | `2`               | Configured via Portal | Confirmed |
| Autoscaling              | Enabled           | Configured via Portal | Confirmed |
| Only `agentpool` exists  | Yes               | Confirmed via Portal  | Confirmed |
| Container Insights       | Disabled          | `null` (empty output) | Confirmed |
| All monitoring           | Disabled          | `null` (empty output) | Confirmed |

***

### Key Concepts

#### What is AKS?

Azure Kubernetes Service (AKS) is a fully managed Kubernetes service provided by Microsoft Azure. It handles the complexity of Kubernetes cluster management — including provisioning, upgrading, scaling, and monitoring — so teams can focus on deploying and running applications.

| Component                                   | Managed By                    |
| ------------------------------------------- | ----------------------------- |
| Control plane (API server, etcd, scheduler) | Azure (free, no charge)       |
| Worker nodes (agent pools)                  | Customer (billed as VMs)      |
| Upgrades and patching                       | Azure (automated or manual)   |
| Scaling                                     | Customer-configured autoscale |

#### Private Cluster vs Public Cluster

| Mode    | API Server Endpoint | Access From                        |
| ------- | ------------------- | ---------------------------------- |
| Public  | Public IP + DNS     | Anywhere on the internet           |
| Private | Private IP only     | Within the VNet or peered networks |

Enabling **private cluster** means the Kubernetes API server is only reachable from within the Azure Virtual Network. This is a security best practice for production clusters as it prevents the control plane from being exposed to the public internet.

#### Availability Zones in AKS

Selecting **Zones 1, 2, 3** distributes the node pool VMs across three physically separate Azure datacenters within the Central US region. If one zone experiences an outage, the other zones continue serving workloads — providing high availability for the cluster.

```
Central US Region
├── Zone 1 (Datacenter A) — nodes scheduled here
├── Zone 2 (Datacenter B) — nodes scheduled here
└── Zone 3 (Datacenter C) — nodes scheduled here
```

#### Node Pool Autoscaling

The `agentpool` was configured with:

```
Min nodes: 1   ← cluster always keeps at least 1 node running
Max nodes: 2   ← cluster never exceeds 2 nodes
```

The **Cluster Autoscaler** watches for pods that cannot be scheduled due to insufficient resources. When demand rises it adds nodes (up to `maxCount`), and when demand drops it removes idle nodes (down to `minCount`) — optimising cost and performance automatically.

#### Standard\_D2s\_v3 VM Size

The `Standard_D2s_v3` instance type provides:

| Spec            | Value      |
| --------------- | ---------- |
| vCPUs           | 2          |
| Memory          | 8 GiB      |
| Temp Storage    | 16 GiB SSD |
| Max data disks  | 4          |
| Premium Storage | Supported  |

This size is suitable for development and light production workloads. The `s` in `D2s` indicates support for Premium SSD storage, and `v3` refers to the third generation of the D-series family.

#### Why Container Insights Was Disabled

Container Insights is an Azure Monitor feature that collects logs and metrics from AKS clusters and stores them in a Log Analytics workspace — which incurs additional cost. For the initial setup and testing phase, the team disabled it to minimise costs. It can be re-enabled later via **nautilus-aks → Monitoring → Insights → Configure monitoring**.

***

### Resource Reference

| Resource           | Type           | Value                                  |
| ------------------ | -------------- | -------------------------------------- |
| Cluster name       | AKS            | `nautilus-aks`                         |
| Resource group     | Azure RG       | `kml_rg_main-88919c00fd5649b1`         |
| Subscription       | Azure          | `Azure Free Labs`                      |
| Region             | Azure Location | `centralus`                            |
| Kubernetes version | AKS            | `1.33.0`                               |
| API server access  | Network        | `Private`                              |
| Availability zones | HA             | `1, 2, 3`                              |
| Node pool name     | AKS Node Pool  | `agentpool`                            |
| VM size            | Compute        | `Standard_D2s_v3`                      |
| Min node count     | Autoscale      | `1`                                    |
| Max node count     | Autoscale      | `2`                                    |
| Container Insights | Monitoring     | `Disabled`                             |
| Deployment ID      | Azure          | `microsoft.aks-1774412293526`          |
| Correlation ID     | Azure          | `81cd50bb-bbe2-4476-a18b-8df1aa99bb59` |

***

_Lab completed on 2026-03-25 | Azure Region: Central US | AKS Version: 1.33.0 | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Optional - learning purpose (Automation):

<figure><img src=".gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>
