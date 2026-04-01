# Day 42: Backup and Delete Azure Storage Blob Container

The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their Azure environment. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their Azure environment.

A private blob container named `xfusion-blob-29475` already exists in the `eastus` region under storage account `xfusionst21199`.

1\) Copy the contents of `xfusion-blob-29475` blob container to the `/opt` directory on the `azure-client` host (the landing host once you load this lab).

2\) Delete the blob container `xfusion-blob-29475` from the storage account.

Use the below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-d0d91986eee34fdc@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-d0d91986eee34fdc@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*=nM                                                                                                                                          |
| Start Time | Sun Mar 22 03:34:26 UTC 2026                                                                                                                       |
| End Time   | Sun Mar 22 04:34:26 UTC 2026                                                                                                                       |

### Objective

The objective of this task is to back up data from an existing Azure Blob Storage container to a local directory on the azure-client host and then delete the container to complete cleanup as part of Azure resource optimization.

***

### Environment Details

* **Storage Account:** xfusionst21199
* **Blob Container:** xfusion-blob-29475
* **Region:** eastus
* **Client Host:** azure-client
* **Target Backup Directory:** /opt

***

### Prerequisites

Azure credentials were retrieved using:

```
showcreds
```

Portal URL: [https://portal.azure.com](https://portal.azure.com)\
Username: kk\_lab\_user\_main-d0d91986eee34fdc@azurefreekmlprod.onmicrosoft.com\
Password: zMTWb=nM

***

### Step 1: Login to Azure

```
az login -u "kk_lab_user_main-d0d91986eee34fdc@azurefreekmlprod.onmicrosoft.com" -p "zMTWb=nM"
```

***

### Step 2: Identify Resource Group of Storage Account

```
az storage account show \
  --name xfusionst21199 \
  --query resourceGroup \
  -o tsv
```

#### Output

```
kml_rg_main-d0d91986eee34fdc
```

***

### Step 3: Retrieve Storage Account Key

```
az storage account keys list \
  --account-name xfusionst21199 \
  --resource-group kml_rg_main-d0d91986eee34fdc \
  --query "[0].value" -o tsv
```

#### Output

```
uxIQm1zbDd/AFZVipNoBs+m1qGzyTXY3aTvMHFe8mXBGYq9o2c5X+SFx/yJjdIqGJnEFoKMBlxtj+AStdTLaOg==
```

***

### Step 4: Download Blob Container to Local Directory

```
az storage blob download-batch \
  --account-name xfusionst21199 \
  --account-key "uxIQm1zbDd/AFZVipNoBs+m1qGzyTXY3aTvMHFe8mXBGYq9o2c5X+SFx/yJjdIqGJnEFoKMBlxtj+AStdTLaOg==" \
  --destination /opt \
  --source xfusion-blob-29475
```

#### Output

```
Finished[#############################################################]  100.0000%
[
  "xfusion.txt"
]
```

***

### Step 5: Verify Downloaded Files

```
ls -l /opt
```

#### Output

```
total 12
-rw-r--r-- 1 root root  445 Mar 22 03:34 creds.json
-rw-r--r-- 1 root root 2749 Mar 22 03:34 showcreds
-rw-r--r-- 1 root root   33 Mar 22 03:40 xfusion.txt
```

***

### Step 6: Delete Blob Container

```
az storage container delete \
  --name xfusion-blob-29475 \
  --account-name xfusionst21199 \
  --account-key "uxIQm1zbDd/AFZVipNoBs+m1qGzyTXY3aTvMHFe8mXBGYq9o2c5X+SFx/yJjdIqGJnEFoKMBlxtj+AStdTLaOg=="
```

#### Output

```
{
  "deleted": true
}
```

***

### Conclusion

The blob container `xfusion-blob-29475` was successfully backed up to the local `/opt` directory on the azure-client host. After verification, the container was deleted from Azure Storage Account `xfusionst21199`, completing the cleanup task successfully.

<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
