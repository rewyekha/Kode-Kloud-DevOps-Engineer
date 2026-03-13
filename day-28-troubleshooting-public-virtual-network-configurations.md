# Day 28: Troubleshooting Public Virtual Network Configurations

The Nautilus DevOps Team deployed an Nginx server on an Azure VM in a public VNet named `xfusion-vnet`. However, the server is still inaccessible from the internet.

As a DevOps team member, complete the following tasks:

1. **Verify VNet Configuration**: Ensure `xfusion-vnet` allows internet access.
2. **Attach Public IP**: A public IP named `xfusion-pip` already exists. Attach this public IP to the VM `xfusion-vm` to make it accessible from the internet.
3. **Ensure Accessibility**: Confirm the VM `xfusion-vm` is accessible on port 80.

Use the provided Azure credentials to troubleshoot and resolve the issue.

Use the following Azure Credentials: (Run `showcreds` on `azure-client` to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-a9a917ab40a24f20@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-a9a917ab40a24f20@azurefreekmlprod.onmicrosoft.com) |
| Password   | =@7P8wWB                                                                                                                                           |
| Start Time | Mon Mar 02 04:07:35 UTC 2026                                                                                                                       |
| End Time   | Mon Mar 02 05:07:35 UTC 2026                                                                                                                       |

\
`Notes:`

* Create resources only in the `East US` region.
* Ensure the Network Security Group (NSG) is attached to the VM's NIC or subnet and configured to allow HTTP traffic on port 80.



***

### 📌 Scenario

The Nautilus DevOps Team deployed an Nginx server on an Azure VM named `xfusion-vm` inside a public VNet `xfusion-vnet`.

However, the server was **not accessible from the internet**.

#### 🎯 Objective

* Verify VNet configuration
* Attach existing Public IP `xfusion-pip`
* Ensure HTTP (Port 80) accessibility
* Confirm Nginx is reachable from browser

***

## 🧠 Troubleshooting Strategy

When a VM is inaccessible from the internet, always verify in this order:

1. ✅ Public IP assigned?
2. ✅ NSG allows inbound traffic?
3. ✅ VNet/Subnet properly configured?
4. ✅ Service running inside VM?

***

## 🏗 Architecture Overview

```
Internet
   │
Public IP (xfusion-pip)
   │
NIC
   │
NSG (Allow 80)
   │
VM (xfusion-vm)
   │
Nginx (Port 80)
```

If any layer blocks traffic → VM becomes unreachable.

***

## 🔎 Step 1: Verify Virtual Network

Navigate to:

**Azure Portal → Virtual Networks → xfusion-vnet**

Confirm:

* Region: **East US**
* Subnet exists
* No custom route blocking 0.0.0.0/0
* No restrictive UDR attached

> 💡 By default, Azure VNets allow outbound internet traffic unless overridden by UDR.

***

## 🌐 Step 2: Attach Public IP to VM

The public IP `xfusion-pip` already existed but was not attached.

### Portal Method

1. Go to **Virtual Machines → xfusion-vm**
2. Click **Networking**
3. Select attached **Network Interface**
4. Open **IP Configurations**
5. Under **Public IP Address**
6. Select **xfusion-pip**
7. Click **Save**

After saving:

VM → Overview → Confirm Public IP is visible.

***

### CLI Alternative

```bash
az network nic ip-config update \
  --name ipconfig1 \
  --nic-name <nic-name> \
  --resource-group <resource-group> \
  --public-ip-address xfusion-pip
```

***

## 🔐 Step 3: Configure Network Security Group (NSG)

This is the most common issue.

### Check NSG Association

Go to:

VM → Networking

Confirm NSG is attached either:

* To NIC\
  OR
* To Subnet

***

### Add Inbound Rule for HTTP

Navigate to:

NSG → Inbound Security Rules → Add

| Setting          | Value      |
| ---------------- | ---------- |
| Source           | Any        |
| Source Port      | \*         |
| Destination      | Any        |
| Protocol         | TCP        |
| Destination Port | 80         |
| Action           | Allow      |
| Priority         | 1000       |
| Name             | allow-http |

Save rule.

***

### CLI Alternative

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name allow-http \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80
```

***

## 🖥 Step 4: Verify Nginx Service

SSH into VM:

```bash
ssh <username>@<public-ip>
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

If not running:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

***

## 🌍 Step 5: Validate Access

Open browser:

```
http://<public-ip>
```

Expected output:

```
Welcome to nginx!
```

***

## 🧪 Validation Using Azure CLI

Check VM public exposure:

```bash
az vm show -g <resource-group> -n xfusion-vm -d --output table
```

Verify:

* Public IP present
* PowerState: running

***

## 🛑 Root Cause Analysis

The issue occurred because:

* The VM did not have a Public IP attached\
  OR
* NSG did not allow inbound TCP port 80

Even if Nginx is running correctly, Azure blocks traffic unless explicitly allowed in NSG.

***

## 🔐 Security Best Practice

Instead of allowing:

```
Source: Any
```

For production environments, restrict to:

* Specific IP ranges
* Azure Application Gateway
* Azure Load Balancer
* Azure Firewall

***

## ✅ Final Checklist

| Check                   | Status |
| ----------------------- | ------ |
| VNet in East US         | ✅      |
| Public IP attached      | ✅      |
| NSG allows TCP 80       | ✅      |
| Nginx running           | ✅      |
| Accessible from browser | ✅      |

***

## 📚 Key Learnings

* Public VNet ≠ Public VM
* Public IP is mandatory for internet access
* NSG rules override everything
* Always troubleshoot layer by layer

***

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

**BEFORE:**\
![](<.gitbook/assets/image (28).png>)

<figure><img src=".gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

**AFTER:**\
![](<.gitbook/assets/image (30).png>)



<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>
