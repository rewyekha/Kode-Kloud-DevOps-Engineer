# Day 38: Running Containers on Azure Virtual Machines

The Nautilus DevOps team needs to set up an Azure Virtual Machine (VM) to interact with an Azure Blob Storage container for storing and retrieving data. The team must create a private storage account, configure Blob Storage, and test the functionality.

#### Task: <a href="#task" id="task"></a>

1\) **Azure Virtual Machine Setup**:

* The VM named `xfusion-vm` already exists in the **East US** region.

2\) **Create a Private Storage Account and Blob Container**:

* Create a storage account named `xfusionstor6577` in the **East US** region with **Locally-redundant storage (LRS)**.
* Create a private Blob container named `xfusion-container6577`.

3\) **Retrieve Storage Account Key**:

* Get the storage account's access key to configure access for the application.

4\) **Create a Test File**:

* SSH into the VM and create a file named `testfile.txt` in the `/home/azureuser` directory with content: "this is a test file".

5\) **Upload the File to Blob Storage**:

*   Upload the `testfile.txt` file to the Blob container `xfusion-container6577` using the Azure CLI command:

    `az storage blob upload --account-name xfusionstor6577 --account-key <access-key> --container-name xfusion-container6577 --name testfile.txt --file /home/azureuser/testfile.txt`



Use below given Azure Credentials: (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

\
`Notes:`

* Create the resources only in the `East US` region.
* Use the Azure Portal or Azure CLI for resource creation.
* Ensure the storage account is private and secure.



***

### Task

**Requirements:**

1. **Azure Virtual Machine Setup**
   * The VM named `xfusion-vm` already exists in the East US region.
2. **Create a Private Storage Account and Blob Container**
   * Create a storage account named `xfusionstor26749` in East US with Locally-redundant storage (LRS).
   * Create a private Blob container named `xfusion-container26749`.
3. **Retrieve Storage Account Key**
   * Obtain the storage account access key to configure access for the application.
4. **Create a Test File**
   * SSH into the VM and create a file named `testfile.txt` in `/home/azureuser` containing: `"this is a test file"`.
5.  **Upload the File to Blob Storage**

    * Upload the file using:

    ```bash
    az storage blob upload \
      --account-name xfusionstor26749 \
      --account-key <access-key> \
      --container-name xfusion-container26749 \
      --name testfile.txt \
      --file /home/azureuser/testfile.txt
    ```

**Azure Credentials Provided:**

| Parameter  | Value                                                                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| Username   | [kk\_lab\_user\_main-d8271c08472a4ee8@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-d8271c08472a4ee8@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*\*                                                                                                                                           |
| Start Time | Wed Mar 18 15:04:18 UTC 2026                                                                                                                       |
| End Time   | Wed Mar 18 16:04:18 UTC 2026                                                                                                                       |

**Notes:**

* All resources must be in **East US**.
* The storage account must be **private and secure**.
* Use **Azure CLI or Portal**.

***

### Step 1: List Resource Groups

```bash
~ ➜ az group list
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b4ee8/resourceGroups/kml_rg_main-d8271c08472a4ee8",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-d8271c08472a4ee8",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

Set environment variables for convenience:

```bash
~ ➜ RESOURCE_GROUP="kml_rg_main-d8271c08472a4ee8"
STORAGE_ACCOUNT="xfusionstor26749"
CONTAINER_NAME="xfusion-container26749"
VM_NAME="xfusion-vm"
FILE_PATH="/home/azureuser/testfile.txt"
LOCATION="eastus"
```

***

### Step 2: Create a Private Storage Account

```bash
~ ➜ az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --allow-blob-public-access false
```

**Terminal Output:**

```json
{
  "accessTier": "Hot",
  "allowBlobPublicAccess": false,
  "creationTime": "2026-03-18T15:09:44.643811+00:00",
  "name": "xfusionstor26749",
  "resourceGroup": "kml_rg_main-d8271c08472a4ee8",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "type": "Microsoft.Storage/storageAccounts"
}
```

***

### Step 3: Create a Private Blob Container

```bash
~ ➜ az storage container create \
  --name $CONTAINER_NAME \
  --account-name $STORAGE_ACCOUNT \
  --auth-mode login \
  --public-access off
```

**Output:**

```json
{
  "created": true
}
```

***

### Step 4: Retrieve the Storage Account Key

```bash
~ ➜ ACCOUNT_KEY=$(az storage account keys list \
  --resource-group $RESOURCE_GROUP \
  --account-name $STORAGE_ACCOUNT \
  --query "[0].value" \
  --output tsv)

echo $ACCOUNT_KEY
```

**Output:**

```
srWkghQ42KyOxKoOquVGyiih7HRnkxsAPzRyb2IMNHfp7WQ37Fpt2+V+EnvT43PsOAWWlurPA1qB+AStjlRjHQ==
```

***

### Step 5: SSH into the VM

```bash
~ ✖ ssh azureuser@52.152.145.125
```

**SSH Output:**

```
The authenticity of host '52.152.145.125 (52.152.145.125)' can't be established.
ECDSA key fingerprint is SHA256:N3LVorl53dJqzSsjmKxJXhRqNBDE+DkiEjpnOCI4eSc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '52.152.145.125' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)

azureuser@xfusion-vm:~$
```

***

### Step 6: Create the Test File

```bash
azureuser@xfusion-vm:~$ echo "this is a test file" > testfile.txt
```

Check Azure CLI version (optional):

```bash
azureuser@xfusion-vm:~$ az --version
```

**Output:**

```
azure-cli                         2.84.0
core                              2.84.0
telemetry                          1.1.0
Python location '/opt/az/bin/python3'
Config directory '/home/azureuser/.azure'
Extensions directory '/home/azureuser/.azure/cliextensions'
Python (Linux) 3.13.11
```

***

### Step 7: Upload the File to Blob Storage

```bash
azureuser@xfusion-vm:~$ az storage blob upload \
  --account-name xfusionstor26749 \
  --account-key srWkghQ42KyOxKoOquVGyiih7HRnkxsAPzRyb2IMNHfp7WQ37Fpt2+V+EnvT43PsOAWWlurPA1qB+AStjlRjHQ== \
  --container-name xfusion-container26749 \
  --name testfile.txt \
  --file /home/azureuser/testfile.txt
```

**Terminal Output:**

```json
{
  "client_request_id": "37b312ce-22dd-11f1-91e2-e5fba1436d47",
  "content_md5": "QiHQAs6108npE35JXOqmRw==",
  "date": "2026-03-18T15:14:53+00:00",
  "etag": "\"0x8DE85011BECAB8A\"",
  "lastModified": "2026-03-18T15:14:54+00:00",
  "request_server_encrypted": true,
  "version": "2026-02-06"
}
```

✅ **Success:** The file `testfile.txt` is now uploaded to the container `xfusion-container26749`.

***

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>
