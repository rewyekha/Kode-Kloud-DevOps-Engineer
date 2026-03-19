# Day 39: Deploying a Static Website Using Containers on Azure

The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on Azure using an Azure Storage account. The Storage account must be configured for public access to allow external users to access the static website directly via the Azure Storage URL.

Task Requirements:

1. Create an Azure Storage account named `xfusionwebst7328` in an existing resource group.
2. Configure the Storage account for static website hosting with `index.html` as the index document.
3. Allow public access to the static website so that the website is publicly accessible.
4. Upload the `index.html` file from the `/root/` directory of the Azure client host to the Storage account's `$web` container.
5. Verify that the website is accessible directly through the Azure Storage static website URL.

Use below given Azure Credentials: (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-1484f06b0b024042@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-1484f06b0b024042@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*\*                                                                                                                                           |
| Start Time | Thu Mar 19 09:32:58 UTC 2026                                                                                                                       |
| End Time   | Thu Mar 19 10:32:58 UTC 2026                                                                                                                       |

\
`Notes:`

* Create the resources only in the `East US` region.
* Use the Azure Storage account's `$web` container to host the static website files.
* To `display` or `hide` the terminal of the Azure client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)





***

## Host a Static Website on Azure Storage

In this guide, we'll cover both the **CLI method** and **GUI method** to create an Azure Storage account, enable static website hosting, upload your `index.html` file, and verify that your website is publicly accessible.

***

### **CLI Method**

#### Step 1: Login to Azure CLI

Before starting, authenticate with your Azure account. Use the following command to log in to Azure CLI:

```bash
az login --username "kk_lab_user_main-1484f06b0b024042@azurefreekmlprod.onmicrosoft.com" --password "*****"
```

This will log you in to your Azure account using the provided credentials.

***

#### Step 2: List Resource Groups

To find the existing resource group where you want to create the storage account, run the following command:

```bash
az group list
```

You should see an output similar to this, confirming that your resource group exists:

```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-1484f06b0b024042",
    "location": "eastus",
    "name": "kml_rg_main-1484f06b0b024042",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

***

#### Step 3: Create the Azure Storage Account

Create the Azure Storage account where you will host your static website:

```bash
az storage account create \
  --name xfusionwebst7328 \
  --resource-group kml_rg_main-1484f06b0b024042 \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot
```

***

#### Step 4: Enable Static Website Hosting

Enable static website hosting on your storage account and set the `index.html` file as the index document:

```bash
az storage blob service-properties update \
  --account-name xfusionwebst7328 \
  --static-website \
  --index-document index.html
```

***

#### Step 5: Upload the `index.html` File

Upload the `index.html` file located at `/root/index.html` to the `$web` container in your storage account:

```bash
az storage blob upload \
  --account-name xfusionwebst7328 \
  --container-name \$web \
  --name index.html \
  --file /root/index.html
```

***

#### Step 6: Verify the Static Website

Once the file is uploaded, verify that the static website is accessible by visiting the following URL:

```plaintext
https://xfusionwebst7328.z13.web.core.windows.net/
```

If everything is set up correctly, your site should display the message **"Welcome to KKE labs!"**.

***

### **GUI Method**

If you prefer to use the Azure Portal (GUI) instead of the Azure CLI, follow the steps below:

#### Step 1: Sign in to the Azure Portal

Go to the [Azure Portal](https://portal.azure.com) and sign in with your Azure account.

***

#### Step 2: Create a New Storage Account

1. **Navigate to the "Storage Accounts" section**:
   * On the left sidebar, click on **"Storage Accounts"** under **"All services"**.
2. **Create a new Storage Account**:
   * Click on the **+ Add** button to create a new storage account.
   * Fill in the following details:
     * **Subscription**: Select your subscription.
     * **Resource Group**: Select the existing resource group `kml_rg_main-1484f06b0b024042`.
     * **Storage Account Name**: Set the name to `xfusionwebst7328`.
     * **Region**: Select **East US**.
     * **Performance**: Standard.
     * **Redundancy**: Locally redundant storage (LRS).
3. Click **Review + Create** and then **Create** to create the storage account.

***

#### Step 3: Enable Static Website Hosting

1. Once the storage account is created, navigate to it by clicking on the **"Storage Accounts"** link in the sidebar.
2. **Select your storage account** (`xfusionwebst7328`).
3. On the **left sidebar**, scroll down and click on **"Static website"** under the **"Data Management"** section.
4. **Enable Static Website** by setting **"Static website"** to **Enabled**.
5. Set **Index document name** to `index.html`.
6. You can leave the **Error document path** empty or set it to `404.html` (optional).
7. Click **Save** to apply the changes.

***

#### Step 4: Upload the `index.html` File

1. Navigate to **"Containers"** in the storage account.
2. Click on the **$web** container.
3. Click **+ Upload**.
4. Select the **index.html** file from your local machine.
5. Click **Upload**.

***

#### Step 5: Verify the Static Website

After uploading the `index.html` file, you can access your static website through the URL:

```plaintext
https://xfusionwebst7328.z13.web.core.windows.net/
```

Your browser should display the message **"Welcome to KKE labs!"**.

***

### Troubleshooting

#### **Missing Credentials Error in CLI Method**

If you encounter the error about missing credentials, you can use the `--auth-mode login` flag to authenticate via Azure CLI:

```bash
az storage blob upload \
  --account-name xfusionwebst7328 \
  --container-name \$web \
  --name index.html \
  --file /root/index.html \
  --auth-mode login
```

This will authenticate the request using your active login credentials.



<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>
