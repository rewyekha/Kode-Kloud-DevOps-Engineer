# Day 14: Create and Attach Managed Disks in Azure

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a managed disk with the following requirements:

* Name of the disk should be `devops-disk`.
* Disk `type` must be `Standard_LRS`.
* Disk `size` must be `2 GiB`.

Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-c2b9210d8af74f50@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-c2b9210d8af74f50@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*                                                                                                                                             |
| Start Time | Fri Jan 09 15:30:03 UTC 2026                                                                                                                       |
| End Time   | Fri Jan 09 16:30:03 UTC 2026                                                                                                                       |

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>



## Solution: Create a Managed Disk in Azure

### Step 1: Log in to Azure Portal

1.  Open the Azure Portal:

    ```
    https://portal.azure.com
    ```
2. Log in using the provided credentials:
   * **Username:** [kk\_lab\_user\_main-c2b9210d8af74f50@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-c2b9210d8af74f50@azurefreekmlprod.onmicrosoft.com)
   * **Password:** hd59x$2c

***

### Step 2: Navigate to Disks

1. In the Azure Portal search bar, type **Disks**.
2. Select **Disks** from the search results.
3. Click on **+ Create** to create a new managed disk.

***

### Step 3: Configure Basic Disk Settings

On the **Basics** tab, configure the following:

* **Subscription:** Select the default subscription
* **Resource Group:** Select an existing resource group (or create a new one if required)
*   **Disk Name:**

    ```
    devops-disk
    ```
* **Region:** Select the required region (same region as related resources, if applicable)
* **Availability Zone:** Leave as default (None), unless specified otherwise

***

### Step 4: Configure Disk Details

Set the disk properties as follows:

* **Source type:** None (Empty disk)
*   **Disk Size (GiB):**

    ```
    2
    ```
*   **Disk SKU:**

    ```
    Standard HDD (Standard_LRS)
    ```
* **OS Type:** None

***

### Step 5: Review and Create the Disk

1. Click **Review + Create**.
2. Verify all configuration details.
3. Click **Create** to provision the managed disk.

***

### Step 6: Verify Disk Creation

1. After deployment completes, navigate back to **Disks**.
2. Confirm that the disk **devops-disk** appears in the list.
3. Verify the following properties:
   * **Disk Name:** devops-disk
   * **Disk Size:** 2 GiB
   * **Disk Type:** Standard\_LRS

***

### Result

The managed disk **devops-disk** has been successfully created in Azure with:

* Disk type: **Standard\_LRS**
* Disk size: **2 GiB**

This disk is now ready to be attached to a virtual machine or used as required for the migration process.
