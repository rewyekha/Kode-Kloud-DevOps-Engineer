# Day 34: Enabling Internet Connectivity for Virtual Machines

### Objective

The Nautilus DevOps team encountered connectivity issues on the Azure VM **xfusion-vm**, preventing package installations. The goal of this lab was to:

1. Investigate the connectivity issue.
2. Implement a solution to restore internet access and package installation capabilities.

***

### Prerequisites

* Access to Azure Portal: [https://portal.azure.com](https://portal.azure.com)
* Credentials:

```
Username: kk_lab_user_main-37b9f45c70fd46e6@azurefreekmlprod.onmicrosoft.com
Password: @RU77A-B
```

* SSH key for VM access located at `/root/.ssh/id_rsa` on the `azure-client` host.
* VM located in **East US** region.

***

### Step 1: Verify Internet Connectivity on the VM

1. SSH into the VM:

```bash
ssh -i /root/.ssh/id_rsa azureuser@<VM_Public_IP>
```

2. Test connectivity:

```bash
ping -c 4 8.8.8.8      # Test raw internet connectivity
ping -c 4 www.microsoft.com  # Test DNS resolution and connectivity
nslookup www.microsoft.com   # Verify DNS
sudo apt update              # Test package repository access
```

**Observation:**

* DNS resolution worked.
* `ping` and `apt update` failed → VM could not reach the internet.

***

### Step 2: Analyze the Network Security Group (NSG)

1. In **Azure Portal**, go to:

```
Virtual Machines → xfusion-vm → Networking → Network Security Group (xfusion-nsg)
```

2. Check **Outbound Security Rules**:

| Priority | Name                  | Action | Notes                                                         |
| -------- | --------------------- | ------ | ------------------------------------------------------------- |
| 200      | Block-All-Outbound    | Deny   | Blocks all outbound traffic.                                  |
| 65000    | AllowVnetOutBound     | Allow  | Only allows traffic inside the VNet.                          |
| 65001    | AllowInternetOutBound | Allow  | Allows internet, but ignored due to lower priority than deny. |

**Root Cause:**

* Outbound `Deny` rule had higher priority than the `AllowInternetOutBound` rule → blocked internet access.

***

### Step 3: Fix NSG Outbound Rules

#### GUI Steps

1. Click on **Block-All-Outbound → Edit**.
2. Either:
   * **Disable or delete** the rule, **or**
   * Keep the rule but **adjust the priority** so that `AllowInternetOutBound` has **higher priority (lower number)**.
3. Example: Set `AllowInternetOutBound` priority = 100.
4. Save the changes.

***

#### Recommended Outbound Rule Configuration

| Priority | Name                  | Protocol | Source | Destination    | Port | Action |
| -------- | --------------------- | -------- | ------ | -------------- | ---- | ------ |
| 100      | AllowInternetOutBound | Any      | Any    | Internet       | \*   | Allow  |
| 200      | AllowVnetOutBound     | Any      | Any    | VirtualNetwork | \*   | Allow  |
| 300+     | Optional DenyAll      | Any      | Any    | Any            | \*   | Deny   |

> **Note:** Lower priority numbers are evaluated first in Azure NSG. Ensure Allow rules for the internet are **higher priority than any deny rules**.

***

### Step 4: Verify the Fix

After updating the NSG:

```bash
ping -c 4 8.8.8.8           # Should succeed
curl -I https://www.microsoft.com  # Should succeed (install curl if needed)
sudo apt update             # Should succeed
```

**Observation:**

* `ping` succeeds.
* `apt update` fetches package lists.
* VM can now install packages normally.

***

### Step 5: Install a Test Package

```bash
sudo apt install curl
curl -I https://www.microsoft.com
```

* Confirms full internet access and package installation capability.

***

### Key Takeaways

* **Inbound rules** control traffic coming into the VM.
* **Outbound rules** control traffic leaving the VM → must allow HTTP/HTTPS for package installs.
* Azure NSG rules are **evaluated by priority**: lower numbers = higher priority.
* Always verify both **NIC-level** and **subnet-level NSGs**.

***

✅ **Lab Completed:** Internet connectivity restored, VM can now install packages.



<figure><img src=".gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

