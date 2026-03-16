# Day 33: Integrating Virtual Machines with Application Load Balancer

## Azure Load Balancer Lab – KodeKloud

### Lab Objective

Set up an **Azure Load Balancer** in front of an **Nginx VM** and verify HTTP traffic flows correctly.

#### Requirements

1. Create a **Standard Public Load Balancer** `xfusion-lb`.
2. Configure a **frontend IP** `xfusion-lb-ip` with a public IP.
3. Create a **backend pool** `xfusion-backend-pool` with the Nginx VM.
4. Create a **health probe** `xfusion-health-probe` on port 80.
5. Create a **load balancing rule** `xfusion-lb-rule` routing HTTP traffic.
6. Allow **port 80** in VM NSG.
7. Deploy resources in **East US** region.
8. Verify Load Balancer public IP serves the Nginx page.

***

### GUI Steps

#### 1️⃣ Login

* Open: [Azure Portal](https://portal.azure.com)
* Username: `kk_lab_user_main-98cd1d23500e4180@azurefreekmlprod.onmicrosoft.com`
* Password: `****`

***

#### 2️⃣ Create Public IP

1. Search **Public IP addresses → Create**
2. Fill:

| Field      | Value         |
| ---------- | ------------- |
| Name       | xfusion-lb-ip |
| Region     | East US       |
| SKU        | Standard      |
| Assignment | Static        |

3. Click **Create**

***

#### 3️⃣ Create Load Balancer

1. Search **Load Balancers → Create**
2. Fill **Basics**:

| Field          | Value                          |
| -------------- | ------------------------------ |
| Name           | xfusion-lb                     |
| Region         | East US                        |
| SKU            | Standard                       |
| Type           | Public                         |
| Tier           | Regional                       |
| Resource Group | kml\_rg\_main-98cd1d23500e4180 |

3. Frontend IP configuration:

| Field      | Value         |
| ---------- | ------------- |
| Name       | xfusion-lb-ip |
| Public IP  | xfusion-lb-ip |
| Assignment | Static        |

4. Backend pool:

| Field             | Value                |
| ----------------- | -------------------- |
| Name              | xfusion-backend-pool |
| Virtual network   | xfusion-vmVNET       |
| Network interface | xfusion-vmvmnic      |

5. Health probe:

| Field    | Value                |
| -------- | -------------------- |
| Name     | xfusion-health-probe |
| Protocol | TCP                  |
| Port     | 80                   |

6. Load Balancer rule:

| Field         | Value                |
| ------------- | -------------------- |
| Name          | xfusion-lb-rule      |
| Frontend IP   | xfusion-lb-ip        |
| Backend Pool  | xfusion-backend-pool |
| Protocol      | TCP                  |
| Frontend Port | 80                   |
| Backend Port  | 80                   |
| Health Probe  | xfusion-health-probe |

7. Outbound rules: Leave default
8. Tags: Optional
9. Click **Review + Create → Create**

***

#### 4️⃣ Update NSG

1. Go to **VM → Networking → NSG**
2. Add inbound rule:

| Field            | Value      |
| ---------------- | ---------- |
| Source           | Any        |
| Source Port      | \*         |
| Destination      | Any        |
| Destination Port | 80         |
| Protocol         | TCP        |
| Action           | Allow      |
| Priority         | 1010       |
| Name             | allow-http |

***

#### 5️⃣ Verify Load Balancer in Browser

* Open **Load Balancer public IP**: `http://48.202.210.78`
* You should see:

```
Welcome to nginx!

If you see this page, the nginx web server is successfully installed and working.
```

***

### CLI Verification Script

```bash
#!/bin/bash

# Variables
LB_NAME="xfusion-lb"
RG_NAME="kml_rg_main-98cd1d23500e4180"
FRONTEND_IP="xfusion-lb-ip"
BACKEND_POOL="xfusion-backend-pool"
HEALTH_PROBE="xfusion-health-probe"
LB_RULE="xfusion-lb-rule"
VM_NAME="xfusion-vm"

echo "===== 1️⃣ Check Load Balancer exists ====="
az network lb show --name $LB_NAME --resource-group $RG_NAME --output table

echo "===== 2️⃣ Check Frontend IP configuration ====="
az network lb frontend-ip list --lb-name $LB_NAME --resource-group $RG_NAME --output table

echo "===== 3️⃣ Check Backend pool and VM attached ====="
az network lb address-pool show --lb-name $LB_NAME --name $BACKEND_POOL --resource-group $RG_NAME --output table

echo "Attached NICs:"
az network nic list --query "[?ipConfigurations[?loadBalancerBackendAddressPools[?id.contains(@,'$BACKEND_POOL')]]].{Name:name}" --output table

echo "===== 4️⃣ Check Health probe ====="
az network lb probe show --lb-name $LB_NAME --name $HEALTH_PROBE --resource-group $RG_NAME --output table

echo "===== 5️⃣ Check Load Balancer rule ====="
az network lb rule show --lb-name $LB_NAME --name $LB_RULE --resource-group $RG_NAME --output table

echo "===== 6️⃣ Check NSG inbound rule for port 80 ====="
NIC_ID=$(az vm show -g $RG_NAME -n $VM_NAME --query "networkProfile.networkInterfaces[0].id" -o tsv)
NIC_NSG=$(az network nic show --ids $NIC_ID --query "networkSecurityGroup.id" -o tsv)
az network nsg rule list --nsg-name $(basename $NIC_NSG) --resource-group $RG_NAME --query "[?destinationPortRange=='80']" --output table

echo "===== 7️⃣ Test Load Balancer public IP ====="
LB_IP=$(az network public-ip show --resource-group $RG_NAME --name $FRONTEND_IP --query "ipAddress" -o tsv)
echo "Load Balancer public IP: $LB_IP"
echo "Test in browser: http://$LB_IP"
```

***

#### Sample CLI Output

```bash
===== 1️⃣ Check Load Balancer exists =====
Location    Name        ProvisioningState    ResourceGroup
eastus      xfusion-lb  Succeeded            kml_rg_main-98cd1d23500e4180

===== 2️⃣ Check Frontend IP configuration =====
Name           PrivateIPAllocationMethod    ProvisioningState
xfusion-lb-ip  Dynamic                      Succeeded

===== 3️⃣ Check Backend pool and VM attached =====
Name                  ProvisioningState
xfusion-backend-pool  Succeeded
Attached NICs:
Name
xfusion-vmVMNic

===== 4️⃣ Check Health probe =====
Name                  Protocol  Port  ProvisioningState
xfusion-health-probe  Tcp       80    Succeeded

===== 5️⃣ Check Load Balancer rule =====
Name             FrontendPort  BackendPort  Protocol  ProvisioningState
xfusion-lb-rule  80           80          Tcp       Succeeded

===== 6️⃣ Check NSG inbound rule for port 80 =====
Name        Access  DestinationPortRange
allow-http  Allow   80

===== 7️⃣ Test Load Balancer public IP =====
Load Balancer public IP: 48.202.210.78
Test in browser: http://48.202.210.78
```

***

GUI:

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>
