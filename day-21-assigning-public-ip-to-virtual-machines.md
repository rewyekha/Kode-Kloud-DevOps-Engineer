# Day 21: Assigning Public IP to Virtual Machines

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>



## Creating an Azure VM with Static Public IP

### Overview

The Nautilus DevOps Team has been tasked with creating a new **Azure Virtual Machine** (`devops-vm`) with a **Static Public IP** (`devops-pip`) in the **Central US** region. This VM will host a new application and must be accessible via SSH using a generated key pair.

This guide documents **terminal commands**, **GUI options**, and **verification steps**.

***

### 1. Generate an SSH Key Pair

On the `azure-client` host, generate a new SSH key pair:

```bash
~ ➜ ssh-keygen
```

**Terminal Output:**

```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:mW2eSyLE6Li4PlL7X9c5B/n24kicv4AVi4PSLwa3D3Q root@azure-client
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|            .    |
|     o . = . +   |
|    . = S E =    |
|  .o . = * B =   |
|=oo....   o ..+o.|
+----[SHA256]-----+
```

Check that the keys are created:

```bash
~ ➜ ls /root/.ssh/
authorized_keys  id_rsa  id_rsa.pub
```

View the public key:

```bash
~ ➜ cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIjgTDtdGqGHgdj2AWLlPsZ7CKovK6o8...
```

This **public key** will be used to enable SSH access to the VM.

***

### 2. Azure Portal: Create the VM (`devops-vm`)

1. **Navigate:** [Azure Portal](https://portal.azure.com)
2. **Create a resource → Virtual Machine**
3. **Basics Tab:**
   * **Subscription:** Your subscription
   * **Resource Group:** Create a new one or use existing
   * **VM Name:** `devops-vm`
   * **Region:** Central US
   * **Availability options:** **No infrastructure redundancy required**
   * **Image:** Ubuntu (latest available, e.g., Ubuntu 24.04 LTS)
   * **Size:** Standard\_B1s
4. **Administrator Account:**
   * **Authentication type:** SSH public key
   * **Username:** `azureuser`
   * **SSH public key:** Paste the content of `/root/.ssh/id_rsa.pub`
5. **Disks Tab:**
   * **OS disk type:** Standard SSD
   * **Size:** 30 GB
6. **Networking Tab:**
   * **Public IP:** Create new
     * **Name:** `devops-pip`
     * **SKU:** Standard ✅
     * **Assignment:** Static ✅
     * **Routing preference:** Internet
     * **Availability zone:** Leave default (No)
7. **Review + Create → Create**

***

### 3. Verify VM and SSH Access

Once the VM is deployed, check the public IP:

```bash
~ ➜ ssh azureuser@<public-ip-address>
```

**Example Terminal Output:**

```
~ ➜ ssh azureuser@20.153.188.14
The authenticity of host '20.153.188.14 (20.153.188.14)' can't be established.
ECDSA key fingerprint is SHA256:F2cJsKkkNtyadHIVZFpR3sdWphIvh4SiElyOE+VF5Pc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.153.188.14' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1017-azure x86_64)
```

Verify disk usage and system information:

```bash
azureuser@devops-vm:~$ df -h
azureuser@devops-vm:~$ free -h
```

Now the VM is:

* **Accessible via SSH**
* **Has a static public IP (`devops-pip`)**
* **Ubuntu OS** with **Standard SSD (30 GB)**
* **Located in Central US**
* **No availability zone selected**

***

### 4. Summary

| Property             | Value                               |
| -------------------- | ----------------------------------- |
| VM Name              | devops-vm                           |
| Region               | Central US                          |
| VM Size              | Standard\_B1s                       |
| OS                   | Ubuntu 24.04 LTS                    |
| Disk                 | Standard SSD, 30 GB                 |
| SSH Access           | Public key (/root/.ssh/id\_rsa.pub) |
| Public IP            | devops-pip (Static, Standard)       |
| Availability options | No                                  |

Your **Azure VM** is now ready to host applications with a **stable, static public IP** and SSH access.
