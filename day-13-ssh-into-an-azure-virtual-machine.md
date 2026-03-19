# Day 13: SSH into an Azure Virtual Machine

The Nautilus DevOps team is working on setting up secure SSH access for their virtual machines in Azure. One of the requirements is to add the SSH public key of the root user from the Azure client host (landing host) to the `nautilus-vm` Azure VM's `authorized_keys` file. This ensures secure and password-less SSH access to the VM.

#### Task Details: <a href="#task-details" id="task-details"></a>

1\) **VM Details**:

* The VM is named `nautilus-vm` and is running in the `West US` region. The default SSH user is `azureuser` — use this user to connect to the VM.
* You need to add the root user's SSH public key from the Azure client host to the `authorized_keys` file of the VM's root user.
* The SSH public key of the root user on the Azure client host is located at `/root/.ssh/id_rsa.pub`.

2\) **Public Key Addition**:

* Copy the public key located at `/root/.ssh/id_rsa.pub` on the Azure client host to the `authorized_keys` file of the root user on `nautilus-vm`.
* Ensure that the proper permissions for the `.ssh` folder and `authorized_keys` file are set on the VM.

3\) **Verification**:

* After adding the public key, make sure that you are able to SSH into the `nautilus-vm` VM as the `root` user from the Azure client host without needing a password.

#### Important Notes: <a href="#important-notes" id="important-notes"></a>

* Ensure that the VM is up and running before attempting to SSH.
* You may need to adjust the firewall or security group rules for the VM to allow SSH access.

Use the following Azure credentials to access the Azure portal:

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-620879b0c7e3420a@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-620879b0c7e3420a@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*                                                                                                                                             |
| Start Time | Fri Jan 09 15:05:51 UTC 2026                                                                                                                       |
| End Time   | Fri Jan 09 16:05:51 UTC 2026                                                                                                                       |

***

## Configure Password-less Root SSH Access to nautilus-vm

### Step 1: Verify VM Availability

* Log in to the Azure Portal.
* Navigate to **Virtual Machines → nautilus-vm**.
* Confirm the VM is **running** and note its **public IP address**.
* Ensure **port 22 (SSH)** is allowed in the VM’s inbound network security rules.

***

### Step 2: SSH into the VM as Default User

From the Azure client (landing host), connect to the VM using the default user:

```bash
ssh azureuser@<nautilus-vm-public-ip>
```

***

### Step 3: Switch to Root User on the VM

Once logged in:

```bash
sudo -i
```

***

### Step 4: Create SSH Directory and Files for Root User

Create the required `.ssh` directory and authorized keys file:

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
touch /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

***

### Step 5: Copy Root Public Key from Azure Client Host

On the **Azure client host**, display the root user’s public key:

```bash
cat /root/.ssh/id_rsa.pub
```

Copy the complete output (single line starting with `ssh-rsa` or `ssh-ed25519`).

***

### Step 6: Add Public Key to Root’s authorized\_keys on VM

On `nautilus-vm` (as root), append the copied key:

```bash
echo "PASTE_ROOT_PUBLIC_KEY_HERE" >> /root/.ssh/authorized_keys
```

Verify the key was added:

```bash
cat /root/.ssh/authorized_keys
```

***

### Step 7: Enable Root SSH Login

Edit the SSH daemon configuration file:

```bash
vi /etc/ssh/sshd_config
```

Ensure the following settings are present and uncommented:

```
PermitRootLogin yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

Restart the SSH service:

```bash
systemctl restart sshd
```

***

### Step 8: Verify Permissions

Check file and directory permissions:

```bash
ls -ld /root /root/.ssh /root/.ssh/authorized_keys
```

Expected permissions:

```
drwx------ root root /root
drwx------ root root /root/.ssh
-rw------- root root /root/.ssh/authorized_keys
```

***

### Step 9: Test Password-less Root Login

Exit the VM:

```bash
exit
```

From the Azure client host, test root SSH access:

```bash
ssh root@<nautilus-vm-public-ip>
```

***

### Step 10: Verification

* SSH login succeeds **without a password prompt**
* Root shell access is granted

***

### Result

The root user’s SSH public key from the Azure client host has been successfully added to the `nautilus-vm`, enabling **secure, password-less root SSH access**.

