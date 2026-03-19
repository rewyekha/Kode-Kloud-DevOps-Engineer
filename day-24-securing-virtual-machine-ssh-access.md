# Day 24: Securing Virtual Machine SSH Access

***

## 🖥️ Azure VM Password-less SSH Setup

***

Create an SSH key on the `azure-client` host (if it does not already exist) and configure password-less SSH access to the `azureuser` account on the `nautilus-vm` in Azure.

Additionally, resolve any Azure policy errors encountered during VM deployment.

***

## Part 1: Generate SSH Key on azure-client (Terminal)

***

### 🔎 1️⃣ Check if SSH Key Already Exists

```bash
ls -al ~/.ssh
```

If you see:

* `id_rsa.pub`
* `id_ed25519.pub`

Then a key already exists.

To view it:

```bash
cat ~/.ssh/id_ed25519.pub
```

If no key exists, generate one.

***

## ✅ 2️⃣ Generate a New SSH Key (if it doesn’t exist)

If no key exists, generate one:

#### 🔐 Recommended (modern + secure):

```bash
ssh-keygen -t ed25519 -C "azureuser@nautilus-vm"
```

If ED25519 isn’t supported:

```bash
ssh-keygen -t rsa -b 4096 -C "azureuser@nautilus-vm"
```

When prompted:

* Press **Enter** to accept default location (`~/.ssh/id_ed25519`)
* Optionally set a passphrase (or press Enter for none)

After generation, verify:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output — you’ll paste this into Azure.

***

## Part 2: Configure SSH in Azure Portal (GUI Method)

***

### 🖥️ Create Virtual Machine

1. Go to **Azure Portal**
2. Click **Create a resource**
3. Select **Virtual Machine**

***

### ⚙️ Basics Tab

* Resource Group: Select existing
* VM Name: `nautilus-vm`
* Region: (Recommended: East US)
* Image: Ubuntu 22.04 LTS
* Size: `Standard_B1s`
* Authentication type: **SSH public key**
* Username: `azureuser`
* SSH public key source: **Use existing public key**

Paste the output of:

```bash
cat ~/.ssh/id_ed25519.pub
```

***

### 💾 Disks Tab (Important – Policy Fix)

To avoid `RequestDisallowedByPolicy` error:

* OS Disk Type → **Standard SSD**
* Ensure disk size is **128 GB or less**
* **30 GB is enough for the lab**

❌ Do NOT select:

* Premium SSD
* Disk size > 128 GB

***

### 🌐 Networking Tab

Use existing VNet if already created.

***

### 🚀 Review + Create

Click **Review + Create**\
Then **Create**

***

## Part 3: Test Password-less SSH

From `azure-client`:

```bash
ssh azureuser@<VM_PUBLIC_IP>
```

If configured correctly:

* No password prompt
* Direct login access

***

## 🛠️ Troubleshooting

***

### ❌ Error: RequestDisallowedByPolicy

Cause:

* Disk size greater than 128 GB
* Premium SSD selected

Solution:

* Change OS disk to **Standard SSD**
* Keep disk ≤ 128 GB
* Use VM size `Standard_B1s`

***

### ❌ Deployment Failed (Provisioning State: Failed)

If VM fails during creation:

1. Go to Resource Group
2. Delete only the failed VM resource
3. Recreate VM with compliant settings

No need to delete VNet / NSG / Public IP.

***

## ✅ Final Verification Checklist

✔ SSH key generated on azure-client\
✔ Public key copied correctly\
✔ Authentication type set to SSH\
✔ Standard SSD selected\
✔ Disk ≤ 128 GB\
✔ VM size Standard\_B1s\
✔ Able to SSH without password

***

## 🎯 Result

Secure, password-less SSH access from:

```
azure-client → nautilus-vm
```

Successfully configured using SSH key authentication.

***

