# Day 18: Copy Data to an Azure Blob Storage Container

The Nautilus DevOps team is presently immersed in data migrations, transferring data from on-premise storage systems to Azure Blob containers. They have recently received some data that they intend to copy to one of the Blob containers.

A Blob container named `devops-blob-17165` already exists in the `East US` region under the storage account `devopsst21353`. Copy the file `/tmp/devops.txt` to the Blob container `devops-blob-17165`.

Use below given Azure Credentials: (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-49de696ec0374a85@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-49de696ec0374a85@azurefreekmlprod.onmicrosoft.com) |
| Password   | C2--$T8$                                                                                                                                           |
| Start Time | Tue Jan 13 17:13:21 UTC 2026                                                                                                                       |
| End Time   | Tue Jan 13 18:13:21 UTC                                                                                                                            |

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>



```bash

~ ➜  az storage blob upload \
  --account-name devopsst21353 \
  --container-name devops-blob-17165 \
  --name devops.txt \
  --file /tmp/devops.txt \
  --auth-mode login
Alive[############################################################Finished[#############################################################]  100.0000%
{
  "client_request_id": "f934ec7c-f0a3-11f0-97b0-1a9147d1146f",
  "content_md5": "Lu7zilatbGguzSz2Ecn5IQ==",
  "date": "2026-01-13T17:19:11+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DE52C7DE72C39B\"",
  "lastModified": "2026-01-13T17:19:11+00:00",
  "request_id": "ecf9bdd4-a01e-004d-45b0-84c122000000",
  "request_server_encrypted": true,
  "version": "2022-11-02",
  "version_id": null
}

~ ➜  az storage blob list \
  --account-name devopsst21353 \
  --container-name devops-blob-17165 \
  --auth-mode login \
  --output table
Name        Blob Type    Blob Tier    Length    Content Type    Last Modified              Snapshot
----------  -----------  -----------  --------  --------------  -------------------------  ----------
devops.txt  BlockBlob    Hot          33        text/plain      2026-01-13T17:19:11+00:00

~ ➜  

```

Sure — here is the **Azure CLI method** to copy `/tmp/devops.txt` to the Blob container.\
These steps are suitable for **lab / terminal execution** ✅

***

### Azure CLI – Upload File to Blob Container

#### 1. Login to Azure using CLI

Run the following command on the **azure-client host**:

```bash
az login
```

* A browser window will open
* Log in using:
  * **Username:** `kk_lab_user_main-49de696ec0374a85@azurefreekmlprod.onmicrosoft.com`
  * **Password:** `C2--$T8$`

After successful login, return to the terminal.

***

#### 2. Set the Correct Subscription (if required)

Check available subscriptions:

```bash
az account list --output table
```

If needed, set the correct one:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

***

#### 3. Upload the File to the Blob Container

Use the following command to upload `/tmp/devops.txt`:

```bash
az storage blob upload \
  --account-name devopsst21353 \
  --container-name devops-blob-17165 \
  --name devops.txt \
  --file /tmp/devops.txt \
  --auth-mode login
```

✅ `--auth-mode login` uses your Azure login credentials (no key required)

***

#### 4. Verify the Upload

List blobs in the container:

```bash
az storage blob list \
  --account-name devopsst21353 \
  --container-name devops-blob-17165 \
  --auth-mode login \
  --output table
```

You should see:

```
Name
---------
devops.txt
```

***

### ✅ Task Completed

* File `/tmp/devops.txt` successfully uploaded
* Storage Account: **devopsst21353**
* Container: **devops-blob-17165**
