# Day 32: Synchronizing Containers Using the CLI

As part of a data migration project, the team lead has tasked the team with migrating data from an existing Azure Blob container to a new Blob container. The existing container contains a substantial amount of data that must be accurately transferred to the new container. The team is responsible for creating the new Blob container and ensuring that all data from the existing container is copied or synced to the new container completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new container without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

Create a New Private Azure Blob Container: Name the container `datacenter-dest-19734` under the storage account `datacenterst26237`.

Data Migration: Migrate the file `datacenter.txt` from the existing `datacenter-source-13818` container to the new `datacenter-dest-19734` container.

Ensure Data Consistency: Ensure that both containers have the file `datacenter.txt` and confirm the file content is identical in both containers.

Use Azure `CLI`: Use the Azure `CLI` to perform the creation and data migration tasks.

***

### Problem Statement

As part of a data migration project, the team lead has tasked the team with migrating data from an existing Azure Blob container to a new Blob container. The existing container contains a substantial amount of data that must be accurately transferred to the new container.

The team is responsible for:

1. Creating the new Blob container.
2. Migrating the required file from the existing container.
3. Verifying that the data is successfully transferred without loss or corruption.

#### Requirements

* Create a **new private Azure Blob container** named `datacenter-dest-19734`.
* The container must be created under the storage account `datacenterst26237`.
* Copy the file `datacenter.txt` from the container `datacenter-source-13818` to the new container.
* Ensure **both containers contain the file**.
* Verify that the **file contents are identical**.
* Use the **Azure CLI**.

Implementation is done using the **Azure CLI**.

***

## Step 1: Define Variables

```bash
STORAGE_ACCOUNT=datacenterst26237
SRC_CONTAINER=datacenter-source-13818
DEST_CONTAINER=datacenter-dest-19734
FILE=datacenter.txt
```

Terminal:

```bash
~ ➜  STORAGE_ACCOUNT=datacenterst26237
SRC_CONTAINER=datacenter-source-13818
DEST_CONTAINER=datacenter-dest-19734
FILE=datacenter.txt
```

***

## Step 2: Create Destination Blob Container

Command:

```bash
az storage container create \
  --name $DEST_CONTAINER \
  --account-name $STORAGE_ACCOUNT \
  --public-access off \
  --auth-mode login
```

Terminal Output:

```bash
~ ➜  az storage container create \
  --name $DEST_CONTAINER \
  --account-name $STORAGE_ACCOUNT \
  --public-access off \
  --auth-mode login
{
  "created": true
}
```

***

## Step 3: Verify Containers

Command:

```bash
az storage container list \
  --account-name $STORAGE_ACCOUNT \
  --output table
```

Terminal Output:

```bash
~ ➜  az storage container list \
  --account-name $STORAGE_ACCOUNT \
  --output table

Name                     Lease Status    Last Modified
-----------------------  --------------  -------------------------
datacenter-dest-19734                    2026-03-12T04:08:16+00:00
datacenter-source-13818                  2026-03-12T04:05:55+00:00
```

***

## Step 4: Copy Blob from Source Container to Destination

Command:

```bash
az storage blob copy start \
  --account-name $STORAGE_ACCOUNT \
  --destination-container $DEST_CONTAINER \
  --destination-blob $FILE \
  --source-container $SRC_CONTAINER \
  --source-blob $FILE \
  --auth-mode login
```

Terminal Output:

```bash
~ ➜  az storage blob copy start \
  --account-name $STORAGE_ACCOUNT \
  --destination-container $DEST_CONTAINER \
  --destination-blob $FILE \
  --source-container $SRC_CONTAINER \
  --source-blob $FILE \
  --auth-mode login
{
  "client_request_id": "25a6db7e-1dc9-11f1-b283-925b92489da9",
  "copy_id": "ff6f52e9-fa58-4585-8d9b-a9e04b9f5c7b",
  "copy_status": "success",
  "date": "2026-03-12T04:08:39+00:00",
  "etag": "\"0x8DE7FED0B194BA5\"",
  "last_modified": "2026-03-12T04:08:40+00:00",
  "request_id": "79e225f2-801e-0033-5fd5-b13414000000",
  "version": "2022-11-02",
  "version_id": null
}
```

***

## Step 5: Verify File in Source Container

Command:

```bash
az storage blob list \
  --account-name $STORAGE_ACCOUNT \
  --container-name $SRC_CONTAINER \
  --output table
```

Terminal Output:

```bash
~ ➜  az storage blob list \
  --account-name $STORAGE_ACCOUNT \
  --container-name $SRC_CONTAINER \
  --output table

Name            Blob Type    Blob Tier    Length    Content Type    Last Modified              Snapshot
--------------  -----------  -----------  --------  --------------  -------------------------  ----------
datacenter.txt  BlockBlob    Hot          33        text/plain      2026-03-12T04:05:59+00:00
```

***

## Step 6: Verify File in Destination Container

Command:

```bash
az storage blob list \
  --account-name $STORAGE_ACCOUNT \
  --container-name $DEST_CONTAINER \
  --output table
```

Terminal Output:

```bash
~ ➜  az storage blob list \
  --account-name $STORAGE_ACCOUNT \
  --container-name $DEST_CONTAINER \
  --output table

Name            Blob Type    Blob Tier    Length    Content Type    Last Modified              Snapshot
--------------  -----------  -----------  --------  --------------  -------------------------  ----------
datacenter.txt  BlockBlob    Hot          33        text/plain      2026-03-12T04:08:40+00:00
```

***

## Step 7: Download Files for Verification

Download from source container:

```bash
az storage blob download \
  --account-name $STORAGE_ACCOUNT \
  --container-name $SRC_CONTAINER \
  --name $FILE \
  --file source.txt \
  --auth-mode login
```

Download from destination container:

```bash
az storage blob download \
  --account-name $STORAGE_ACCOUNT \
  --container-name $DEST_CONTAINER \
  --name $FILE \
  --file dest.txt \
  --auth-mode login
```

Terminal Output:

```bash
Finished[#############################################################]  100.0000%
```

***

## Step 8: Compare File Contents

Command:

```bash
diff source.txt dest.txt
```

Terminal Output:

```bash
~ ➜  diff source.txt dest.txt
```

No output is returned, which means both files are **identical**.

***

## Final Verification

| Validation                     | Result |
| ------------------------------ | ------ |
| Destination container created  | ✅      |
| File copied successfully       | ✅      |
| File exists in both containers | ✅      |
| File size identical            | ✅      |
| File content identical         | ✅      |

***

## Conclusion

The file `datacenter.txt` was successfully migrated from the source container `datacenter-source-13818` to the destination container `datacenter-dest-19734` using the **Azure CLI**. Verification confirmed that the file exists in both containers and the contents are identical, ensuring a successful and consistent data migration.

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
