# Day 16: Create a Private Azure Blob Storage Container

As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize private Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named `xfusionst20516` and a `private` Blob container named `xfusion-blob-20022` within the storage account.

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-96d89b69340b473b@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-96d89b69340b473b@azurefreekmlprod.onmicrosoft.com) |
| Password   | TJEb+cTK                                                                                                                                           |
| Start Time | Mon Jan 12 17:00:49 UTC 2026                                                                                                                       |
| End Time   | Mon Jan 12 18:00:49 UTC 2026                                                                                                                       |

You’re **almost done** 👍\
The error is just because `<STORAGE_ACCOUNT_KEY>` is a **placeholder**, not a real value.

You already retrieved the **actual storage account key**. You must paste it **directly** or store it in a variable.

***

### ✅ Option 1 (Recommended): Use a Variable

Run this **exactly**:

```bash
STORAGE_KEY=$(az storage account keys list \
  --resource-group kml_rg_main-96d89b69340b473b \
  --account-name xfusionst20516 \
  --query "[0].value" \
  --output tsv)
```

Now create the container:

```bash
az storage container create \
  --name xfusion-blob-20022 \
  --account-name xfusionst20516 \
  --account-key "$STORAGE_KEY" \
  --public-access off
```

***

### ✅ Option 2: Paste the Key Directly (Quick Fix)

```bash
az storage container create \
  --name xfusion-blob-20022 \
  --account-name xfusionst20516 \
  --account-key WqJsK8L2tp1V/cokc1lhmsJCfu/ZU8KXSrqFnEWB7D2oW4QSBF5CDNpb/h9m5fNfGLnkxnYgFbnD+AStYZUs7Q== \
  --public-access off
```

***

### 🔍 Verify the Container

```bash
az storage container show \
  --name xfusion-blob-20022 \
  --account-name xfusionst20516 \
  --account-key "$STORAGE_KEY"
```

***

#### ✅ Final Status

* ✔ Storage Account: **xfusionst20516**
* ✔ Blob Container: **xfusion-blob-20022**
* ✔ Access Level: **Private**

\
GUI:<br>

### Step 1: Sign in to Azure Portal

1. Open 👉 [**https://portal.azure.com**](https://portal.azure.com)
2. Sign in with:
   * **Username:**\
     `kk_lab_user_main-96d89b69340b473b@azurefreekmlprod.onmicrosoft.com`
   * **Password:**\
     `TJEb+cTK`

***

### Step 2: Create the Storage Account

1. In the Azure Portal search bar, type **Storage accounts**
2. Click **Storage accounts**
3. Click **+ Create**

#### Basics tab

Fill in the following:

* **Subscription:** Default
* **Resource group:**
  * Select an existing resource group
  * OR click **Create new** (for example: `xfusion-rg`)
* **Storage account name:**\
  `xfusionst20516`
* **Region:** Choose the required region (for example **East US**)
* **Performance:** Standard
* **Redundancy:** Locally-redundant storage (LRS)

4. Click **Review + create**
5. Click **Create**

✅ Storage account **xfusionst20516** is now created.

***

### Step 3: Open the Storage Account

1. Once deployment completes, click **Go to resource**
2. You should now be inside **xfusionst20516**

***

### Step 4: Create the Private Blob Container

1. In the left menu, scroll to **Data storage**
2. Click **Containers**
3. Click **+ Container**

Fill in:

* **Name:**\
  `xfusion-blob-20022`
* **Public access level:**\
  **Private (no anonymous access)**

4. Click **Create**

***

### Step 5: Verify

Under **Containers**, confirm:

* Container **xfusion-blob-20022**
* **Public access level:** Private

***

#### ✅ Final Result

* **Storage Account:** `xfusionst20516`
* **Blob Container:** `xfusion-blob-20022`
* **Access Level:** Private



