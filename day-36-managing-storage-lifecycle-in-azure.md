# Day 36: Managing Storage Lifecycle in Azure

The Nautilus DevOps team needs to optimize data retention costs by automating the deletion of old blobs. They plan to implement Blob Lifecycle Management for a specific container in Azure Storage.

#### Task: <a href="#task" id="task"></a>

1\) **Create a Storage Account**:

* Name the storage account `nautilusstor1846`.
* Set the **region** to **East US**.
* Use **Locally-redundant storage (LRS)** as the redundancy option.

2\) **Create a Blob Container**:

* Name the container `nautilus-container1846`.

3\) **Upload a File to the Container**:

* Upload the file named `tempfile.txt` to the container. The file is present under `/root` of the client host.

4\) **Configure Blob Lifecycle Management**:

* Apply a Lifecycle Management rule named `nautilus-del-rule` to the container `nautilus-container1846` to delete blobs after `7` days of last modification.

5\) **Validation**:

* Verify that the Lifecycle Management rule named `nautilus-del-rule` is correctly applied.

\
`Notes:`

* Create the resources only in the `East US` region.
* Use the Azure Portal or Azure CLI for resource creation.
* Ensure the storage account and container are properly configured.



**Solution steps:**

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>



```bash

~ ➜  STORAGE_ACCOUNT="nautilusstor1846"
CONTAINER_NAME="nautilus-container1846"
FILE_PATH="/root/tempfile.txt"
BLOB_NAME="tempfile.txt"

~ ➜  az storage account keys list \
 --account-name $STORAGE_ACCOUNT \
 --query "[0].value" \
 --output tsv
FYt7QfZV1S6hS71BUI1P8wUT/tBMdV4X9ViCYJL7UCogr+eao9jVpguWiDqQ8eZPIZ1P8cxxvchr+ASty2kW0w==

~ ➜  export AZURE_STORAGE_KEY=FYt7QfZV1S6hS71BUI1P8wUT/tBMdV4X9ViCYJL7UCogr+eao9jVpguWiDqQ8eZPIZ1P8cxxvchr+ASty2kW0w==

~ ➜  az storage blob list \
 --account-name $STORAGE_ACCOUNT \
 --container-name $CONTAINER_NAME \
 --output table \
 --auth-mode key


~ ➜  az storage blob upload \
 --account-name $STORAGE_ACCOUNT \
 --container-name $CONTAINER_NAME \
 --name $BLOB_NAME \
 --file $FILE_PATH \
 --auth-mode key
Finished[#############################################################]  100.0000%
{
  "client_request_id": "ba96246c-214b-11f1-b08e-d673c18f074d",
  "content_md5": "KK8IdCQ+hAnGosayoGGjvA==",
  "date": "2026-03-16T15:20:55+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DE836F9F126E39\"",
  "lastModified": "2026-03-16T15:20:56+00:00",
  "request_id": "5aea854a-901e-002f-6158-b56674000000",
  "request_server_encrypted": true,
  "version": "2022-11-02",
  "version_id": null
}

~ ➜  az storage blob list \
 --account-name $STORAGE_ACCOUNT \
 --container-name $CONTAINER_NAME \
 --output table \
 --auth-mode key
Name          Blob Type    Blob Tier    Length    Content Type    Last Modified              Snapshot
------------  -----------  -----------  --------  --------------  -------------------------  ----------
tempfile.txt  BlockBlob    Hot          25        text/plain      2026-03-16T15:20:56+00:00

```



<figure><img src=".gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
