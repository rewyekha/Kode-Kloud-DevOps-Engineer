---
description: Nautilus DevOps Migration Task
---

# Day 1 - Azure SSH Key Creation

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### Problem Statement

The Nautilus DevOps team is planning a phased migration of a portion of their infrastructure to the Azure cloud. To reduce risk and improve manageability, the team has decided to break the migration into smaller, incremental tasks rather than performing a single large-scale move.

As part of this strategy, the team needs to prepare secure access for future Azure virtual machines by creating an SSH key pair.

#### Task Requirements

* Create an SSH key pair in Azure
* SSH key name must be **nautilus-kp**
* Key pair type must be **RSA**
* Use the provided Azure credentials

***

### Solution

The SSH key pair was created using the Azure CLI on the provided `azure-client` host.

#### Step 1: Retrieve Azure Credentials

Run the following command to display Azure login credentials:

```bash
showcreds
```

Use the provided username and password for authentication.

***

#### Step 2: Login to Azure

Authenticate using device login:

```bash
az login
```

Follow the on-screen instructions:

* Open the provided URL in a browser
* Enter the device code
* Login with the Azure username and password

Verify login:

```bash
az account show
```

***

#### Step 3: Identify Resource Group and Location

List available resource groups:

```bash
az group list -o table
```

Identified values:

* **Resource Group:** `kml_rg_main-bc19af0667854824`
* **Location:** `westus`

***

#### Step 4: Create the SSH Key Pair

Run the following command to create the SSH key:

```bash
az sshkey create \
  --name nautilus-kp \
  --resource-group kml_rg_main-bc19af0667854824 \
  --location westus
```

> **Note:** Azure CLI generates **RSA keys by default**, which satisfies the task requirement.

***

#### Step 5: Verify the SSH Key

Confirm the key creation:

```bash
az sshkey show \
  --name nautilus-kp \
  --resource-group kml_rg_main-bc19af0667854824
```

The output confirms:

* SSH key name: `nautilus-kp`
* Public key starts with `ssh-rsa`, verifying RSA type

***

### Key Storage Details

* **Public Key:** Stored securely in Azure as an SSH key resource
* **Private Key:** Generated and stored locally on the client machine

Example paths:

```
/root/.ssh/<generated_key>
/root/.ssh/<generated_key>.pub
```

> Azure does **not** store or display private keys.

***

### Azure Portal Verification (Optional)

The SSH key can be viewed in the Azure Portal:

1. Open [https://portal.azure.com](https://portal.azure.com/)
2. Search for **SSH keys**
3. Select **nautilus-kp**
4. View key details and public key

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

***

### Conclusion

The SSH key pair **nautilus-kp** was successfully created in Azure using RSA encryption. This completes the required task and prepares the environment for secure, phased infrastructure migration.

