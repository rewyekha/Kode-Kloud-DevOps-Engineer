# Day 25: Expanding and Managing Disk Storage

***

## 📘 Datacenter VM Disk Expansion & Data Disk Mount

### **Objective**

* Resize the OS disk of `datacenter-vm` from 32 GiB → 64 GiB.
* Add a new 64 GiB data disk and mount it at `/mnt/datacenter-disk`.

***

### **1️⃣ Azure CLI: Resize OS Disk**

#### **Step 1: Check VM and Disk Details**

```bash
az vm list -o table
```

```
Name           ResourceGroup                 Location
-------------  ----------------------------  ----------
datacenter-vm  KML_RG_MAIN-8760574443A04B6D  eastus
```

```bash
az vm show -g KML_RG_MAIN-8760574443A04B6D -n datacenter-vm --query "storageProfile.osDisk.name" -o tsv
```

```
datacenter-vm_disk1_ea5a70be788242978fd4b6842efcbb8b
```

***

#### **Step 2: Attempt to Resize While VM Running**

```bash
az disk update --resource-group KML_RG_MAIN-8760574443A04B6D --name datacenter-vm_disk1_ea5a70be788242978fd4b6842efcbb8b --size-gb 64
```

**Error Encountered:**

```
(OperationNotAllowed) Change in disk property of OS disk '' is not allowed when VM is running.
Code: OperationNotAllowed
Message: Change in disk property of OS disk '' is not allowed when VM is running.
```

**Cause:**

* Azure does not allow resizing of the OS disk while the VM is running.

**Solution:**

* Deallocate the VM before resizing the OS disk.

***

#### **Step 3: Deallocate VM**

```bash
az vm deallocate --resource-group KML_RG_MAIN-8760574443A04B6D --name datacenter-vm
```

```bash
az vm get-instance-view --resource-group KML_RG_MAIN-8760574443A04B6D --name datacenter-vm --query "instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus" -o tsv
```

```
VM deallocated
```

***

#### **Step 4: Resize OS Disk**

```bash
az disk update \
  --resource-group KML_RG_MAIN-8760574443A04B6D \
  --name datacenter-vm_disk1_ea5a70be788242978fd4b6842efcbb8b \
  --size-gb 64
```

> Disk resized successfully to 64 GiB.

***

#### **Step 5: Start VM**

```bash
az vm start --resource-group KML_RG_MAIN-8760574443A04B6D --name datacenter-vm
```

***

### **2️⃣ Azure CLI: Create & Attach New Data Disk**

#### **Step 1: Create Data Disk**

```bash
az disk create \
  --resource-group KML_RG_MAIN-8760574443A04B6D \
  --name datacenter-disk \
  --size-gb 64 \
  --sku Standard_LRS
```

> Disk `datacenter-disk` created successfully.

***

#### **Step 2: Attach Disk to VM**

```bash
az vm disk attach \
  --resource-group KML_RG_MAIN-8760574443A04B6D \
  --vm-name datacenter-vm \
  --name datacenter-disk
```

**Note:**

* The `--disk` option is deprecated; `--name` is recommended.

***

### **3️⃣ Linux VM: Partition, Format & Mount**

#### **Step 1: SSH into VM**

```bash
ssh azureuser@23.101.132.31
```

**Error You Might See Initially:**

```
ssh: Could not resolve hostname : Name or service not known
```

**Cause:**

* Command was malformed or missing hostname.

**Solution:**

* Ensure correct syntax: `ssh azureuser@<public-ip>`.

***

#### **Step 2: Verify Disks**

```bash
sudo lsblk
```

```
sda       64G  /  
sdb       4G   /mnt  
sdc       64G  (new disk)
```

***

#### **Step 3: Partition New Disk**

```bash
sudo parted /dev/sdc mklabel gpt
sudo parted /dev/sdc mkpart primary ext4 0% 100%
```

> Partition created successfully.

***

#### **Step 4: Format Partition**

```bash
sudo mkfs.ext4 /dev/sdc1
```

> Filesystem UUID generated, e.g., `48f73eef-2b17-45a8-bb8e-688d45a8ca0d`.

***

#### **Step 5: Mount Disk**

```bash
sudo mkdir -p /mnt/datacenter-disk
sudo mount /dev/sdc1 /mnt/datacenter-disk
```

***

#### **Step 6: Update fstab for Persistent Mount**

```bash
sudo blkid /dev/sdc1
```

```
/dev/sdc1: UUID="48f73eef-2b17-45a8-bb8e-688d45a8ca0d" TYPE="ext4"
```

Edit `/etc/fstab`:

```
UUID=48f73eef-2b17-45a8-bb8e-688d45a8ca0d /mnt/datacenter-disk ext4 defaults,nofail 0 2
```

```bash
sudo mount -a
df -h /mnt/datacenter-disk
```

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc1        63G   24K   60G   1% /mnt/datacenter-disk
```

***

### **4️⃣ Summary of Errors & Solutions**

| Error                                         | Cause                    | Solution                        |
| --------------------------------------------- | ------------------------ | ------------------------------- |
| `(OperationNotAllowed)` when resizing OS disk | VM running               | Deallocate VM before resizing   |
| `ssh: Could not resolve hostname`             | Incorrect SSH syntax     | Use `ssh azureuser@<public-ip>` |
| Disk not mounted after reboot                 | `/etc/fstab` not updated | Add UUID entry to `/etc/fstab`  |

***

### **✅ Result**

* OS disk resized → 64 GiB.
* New 64 GiB data disk attached and mounted at `/mnt/datacenter-disk`.
* Mount persists after reboot.

***

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

