# Day 2 - creating an Azure Virtual Machine (VM)

The Nautilus DevOps team is planning to migrate a portion of their infrastructure to the Azure cloud incrementally. As part of this migration, you are tasked with creating an Azure Virtual Machine (VM).

The requirements are:

1\) Use the existing resource group.

2\) The VM name must be `xfusion-vm`, it should be in `West US` region.

3\) Use the `Ubuntu 22.04 LTS` image for the VM.

4\) The VM size must be `Standard_B1s`.

5\) Attach a default Network Security Group (NSG) that allows inbound SSH (port 22).

6\) Attach a 30 GB storage disk of type `Standard HDD`.

7\) The rest of the configurations should remain as default.

After completing these steps, make sure you can SSH into the virtual machine.



Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-9e3742f0bc2e45ca@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-9e3742f0bc2e45ca@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*                                                                                                                                             |



### ✅ Option 1 — Using the **Azure CLI**

You can run a single command that creates the VM along with the **default Network Security Group (NSG) with SSH allowed**, the **Ubuntu 22.04 LTS image**, **Standard\_B1s size**, and a **30 GB Standard HDD disk**.

> The Azure CLI supports creating a VM with those options in one command, and by default will create the supporting network resources (VNet, NIC, NSG, Public IP) for you. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-cli?utm_source=chatgpt.com))

#### Step-by-Step

1.  **Log in with your Azure credentials**\
    Open the Azure CLI (Cloud Shell or local CLI) and run:

    ```bash
    az login
    ```
2.  **Set variables**\
    Replace `<RESOURCE_GROUP>` with the existing resource group name you must use:

    ```bash
    RESOURCE_GROUP="<RESOURCE_GROUP>"
    VM_NAME="xfusion-vm"
    LOCATION="westus"
    ADMIN_USER="azureuser"
    ```
3.  **Create the VM**

    ```bash
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name $VM_NAME \
      --image Ubuntu2204 \
      --size Standard_B1s \
      --location $LOCATION \
      --admin-username $ADMIN_USER \
      --generate-ssh-keys \
      --nsg-rule SSH \
      --os-disk-size-gb 30 \
      --storage-sku StandardHDD_LRS
    ```

    🔹 **What this does:**

    * `Ubuntu2204` ensures **Ubuntu 22.04 LTS image** is used. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/create-cli-complete?utm_source=chatgpt.com))
    * `Standard_B1s` matches the requested VM size. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-cli?utm_source=chatgpt.com))
    * `--nsg-rule SSH` automatically creates an NSG rule that allows **port 22 inbound** for SSH. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-cli?utm_source=chatgpt.com))
    * `--generate-ssh-keys` will generate SSH key files if you don’t already have them.
    * `--os-disk-size-gb 30` specifies a **30 GB OS disk**.
    * `--storage-sku StandardHDD_LRS` sets the disk type to **Standard HDD**.
4. **Note the IP address**\
   After deployment completes, the command will output the VM’s **public IP address**. You’ll use this to SSH into the VM.
5.  **SSH into the VM**

    ```bash
    ssh azureuser@<PUBLIC_IP_ADDRESS>
    ```

    Use the SSH key that the CLI created (usually in `~/.ssh`).\
    &#xNAN;_(Optional: if you prefer password authentication instead of SSH keys, you can change `--generate-ssh-keys` to `--authentication-type password` and specify a password — but SSH keys are recommended.)_

***

### ✅ Option 2 — Using the **Azure Portal**

If you prefer a UI approach:

1. Sign in at the Azure portal: [https://portal.azure.com](https://portal.azure.com)
2. Search for **Virtual machines** and click **Create > Azure virtual machine**.
3. In **Basics**:
   * **Subscription**: choose your subscription
   * **Resource group**: select the existing one
   * **VM name**: `xfusion-vm`
   * **Region**: `West US`
   * **Image**: **Ubuntu Server 22.04 LTS**
   * **Size**: **Standard\_B1s**
4. Under **Administrator account** choose SSH public key (recommended) and provide your SSH public key.
5. In **Inbound port rules**, **allow SSH (22)**. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal?utm_source=chatgpt.com))
6. Go to the **Disks** tab:
   * Set **OS disk type** to **Standard HDD**
   * Set **OS disk size** to **30 GB**\
     (Defaults are often 30 GB, but ensure SSD isn’t selected.) ([powershellgeek.com](https://www.powershellgeek.com/2020/04/08/azure-virtual-machine-creation-process/?utm_source=chatgpt.com))
7. Leave other tabs at defaults and click **Review + create**, then **Create**.
8. #### ![](<.gitbook/assets/image (21).png>)
9.

    <figure><img src=".gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>
10. When creation finishes, find the **public IP address** on the VM’s **Overview page**.
11. **SSH into the VM** using:

    ```bash
    ssh <your admin username>@<public-ip>
    ```

***

### 📌 Additional Notes

* Azure automatically creates the **NSG with an SSH rule** if you choose **SSH** or use `--nsg-rule SSH` in the CLI. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-cli?utm_source=chatgpt.com))
* The CLI will also create the **VNet, subnet, NIC, and public IP** needed to access the VM. ([Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/create-cli-complete?utm_source=chatgpt.com))
* If you need more granular control (specific network, custom NSG, etc.), you can create the resources separately before the VM.

***

### 🎯 Once Deployed

To SSH:

```bash
ssh azureuser@<PUBLIC_IP>
```

Where `<PUBLIC_IP>` is the VM’s output public IP address.
