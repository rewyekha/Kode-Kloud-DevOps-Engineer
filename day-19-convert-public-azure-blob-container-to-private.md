# Day 19: Convert Public Azure Blob Container to Private

The Nautilus DevOps team has been using Azure Blob Storage to manage their data. Recently, they realized that one of their containers, currently public, needs to be restricted for internal use only. Your task is to convert a public Azure Blob container to private.

Two blob containers named `xfusion-container-5146` and `xfusion-priv-7653` are available in the `East US` region within the storage account `xfusionst23889`. The `xfusion-container-5146` is currently public, and `xfusion-priv-7653` is private.

1\) Convert the blob container `xfusion-container-5146` from public to private while leaving `xfusion-priv-7653` unchanged.

2\) Make sure the access level for `xfusion-container-5146` is set to `private` with no public access.

Use the below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-a5ee41df6fd34900@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-a5ee41df6fd34900@azurefreekmlprod.onmicrosoft.com) |
| Password   | %xxnrXcG                                                                                                                                           |
| Start Time | Tue Jan 13 17:23:17 UTC 2026                                                                                                                       |
| End Time   | Tue Jan 13 18:23:17 UTC 2026                                                                                                                       |

\
`Notes:`

* Perform all operations in the `East US` region.
* The storage account name is `xfusionst23889`.

```bash

~ ➜  az storage container show \
  --name xfusion-container-5146 \
  --account-name xfusionst23889 \
  --auth-mode login
{
  "deleted": null,
  "encryptionScope": {
    "defaultEncryptionScope": "$account-encryption-key",
    "preventEncryptionScopeOverride": false
  },
  "immutableStorageWithVersioningEnabled": false,
  "metadata": {},
  "name": "xfusion-container-5146",
  "properties": {
    "etag": "\"0x8DE52C8A485EDB7\"",
    "hasImmutabilityPolicy": false,
    "hasLegalHold": false,
    "lastModified": "2026-01-13T17:24:44+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "publicAccess": "container"
  },
  "version": null
}

~ ➜  az storage container set-permission \
  --name xfusion-container-5146 \
  --account-name xfusionst23889 \
  --public-access off \
  --auth-mode login
az storage container set-permission: 'login' is not a valid value for '--auth-mode'. Allowed values: key.

Examples from AI knowledge base:
https://aka.ms/cli_ref
Read more about the command in reference docs

~ ✖ az storage account keys list \
  --account-name xfusionst23889 \
  --query "[0].value" \
  --output tsv
rBpIV9488QRa1NCyg1qLCB7q29R/T8TVsTdVhmic5mCt5OV53M963z/8vnsTLX3eJvgJkk6ADzAZ+AStJOmVEQ==

~ ➜  az storage container set-permission \
  --name xfusion-container-5146 \
  --account-name xfusionst23889 \
  --account-key rBpIV9488QRa1NCyg1qLCB7q29R/T8TVsTdVhmic5mCt5OV53M963z/8vnsTLX3eJvgJkk6ADzAZ+AStJOmVEQ== \
  --public-access off
{
  "client_request_id": "4f8ffb4c-f0a5-11f0-868f-9652e486138c",
  "date": "2026-01-13T17:28:44+00:00",
  "etag": "\"0x8DE52C933CA8AD0\"",
  "last_modified": "2026-01-13T17:28:44+00:00",
  "request_id": "56d8c43b-801e-00b5-3eb2-84f8ad000000",
  "version": "2022-11-02"
}

~ ➜  az storage container show \
  --name xfusion-container-5146 \
  --account-name xfusionst23889 \
  --account-key <PASTE_STORAGE_KEY_HERE> \
  --query properties.publicAccess
-bash: PASTE_STORAGE_KEY_HERE: No such file or directory

~ ✖ az storage container show   --name xfusion-container-5146   --account-name xfusionst23889   --account-key rBpIV9488QRa1NCyg1qLCB7q29R/T8TVsTdVhmic5mCt5OV53M963z/8vnsTLX3eJvgJkk6ADzAZ+AStJOmVEQ==  --query prope
rties.publicAccess

~ ➜  
```

### ✅ Correct Way: Use Storage Account Key

#### 1. Get the Storage Account Key

Run:

```bash
az storage account keys list \
  --account-name xfusionst23889 \
  --query "[0].value" \
  --output tsv
```

👉 Copy the key value returned.

***

#### 2. Set the Container to Private

Use the key from above:

```bash
az storage container set-permission \
  --name xfusion-container-5146 \
  --account-name xfusionst23889 \
  --account-key <PASTE_STORAGE_KEY_HERE> \
  --public-access off
```

✅ This removes **all public access** and makes the container private.

***

#### 3. Verify the Access Level

Run:

```bash
az storage container show \
  --name xfusion-container-5146 \
  --account-name xfusionst23889 \
  --account-key <PASTE_STORAGE_KEY_HERE> \
  --query properties.publicAccess
```

Expected output:

```
null
```

(or empty output)

***

### 🔒 Final State (As Required)

| Container              | Public Access                  |
| ---------------------- | ------------------------------ |
| xfusion-container-5146 | **Private (No public access)** |
| xfusion-priv-7653      | Private (unchanged)            |

***

#### 💡 Why this happened

* **Data-plane write operations** like `set-permission` still require:
  * Storage account key **or**
  * SAS token
* `--auth-mode login` works for **read/list**, not permission changes.

Here’s the **Azure Portal (GUI) step-by-step guide** to convert the container from **public → private**, while leaving the other container unchanged.

***

### Azure Portal (GUI) – Make Blob Container Private

#### 1. Sign in to Azure Portal

1. Open [**https://portal.azure.com**](https://portal.azure.com)
2. Log in with:
   * **Username:** `kk_lab_user_main-a5ee41df6fd34900@azurefreekmlprod.onmicrosoft.com`
   * **Password:** `%xxnrXcG`

#### 2. Open the Storage Account

1. In the top **Search bar**, type **Storage accounts**
2. Click **Storage accounts**
3. Select **`xfusionst23889`**\
   (Region: East US)

***

#### 3. Go to Containers

1. In the left menu under **Data storage**, click **Containers**
2. You will see:
   * `xfusion-container-5146` (public)
   * `xfusion-priv-7653` (private)

***

#### 4. Change `xfusion-container-5146` to Private

1. Click **`xfusion-container-5146`**
2. At the top menu, click **Change access level**
3. In the popup:
   * **Public access level** → select **Private (no anonymous access)**
4. Click **OK / Save**

✅ This immediately removes public access.

***

#### 5. Do NOT Modify the Other Container

* **Do nothing** to `xfusion-priv-7653`
* It remains private as required

***

#### 6. Verify (GUI)

1. Still inside **Containers**
2. Look at the **Public access level** column
3. Confirm:
   * `xfusion-container-5146` → **Private**
   * `xfusion-priv-7653` → **Private**

***

### ✅ Final Result

| Container Name         | Access Level                   |
| ---------------------- | ------------------------------ |
| xfusion-container-5146 | **Private (no public access)** |
| xfusion-priv-7653      | Private (unchanged)            |
