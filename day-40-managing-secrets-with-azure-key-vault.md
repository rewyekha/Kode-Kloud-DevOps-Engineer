# Day 40: Managing Secrets with Azure Key Vault

### Task

The Nautilus DevOps team is focusing on improving their data security by using Azure Key Vault. Your task is to create a Key Vault with an RSA key and manage the encryption and decryption of a pre-existing sensitive file using this key.

***

### Requirements

* Create a Key Vault:
  * Name: `nautilus-3581`
  * Region: East US
  * Pricing tier: Standard
  * Soft Delete retention: 7 days
  * Use Vault access policy permission model
  * Grant access policy to lab identity with permissions:
    * Get
    * List
    * Encrypt
    * Decrypt

***

* Create an RSA Key:
  * Name: `nautilus-key`
  * Key type: RSA
  * Key size: 4096
  * Leave all other settings as default

***

* Encrypt the Sensitive Data:
  * File location: `/root/SensitiveData.txt`
  * Use RSA-OAEP algorithm
  * Base64 encode the file before encryption
  * Save encrypted output as: `/root/EncryptedData.bin`

***

* Decrypt the File:
  * Decrypt `/root/EncryptedData.bin`
  * Base64 decode the decrypted output
  * Save final file as `/root/DecryptedData.txt`

***

* Verification:
  * Ensure `/root/DecryptedData.txt` matches `/root/SensitiveData.txt`

***

### Notes

*   If you encounter permission errors, retrieve Service Principal ID using:

    ```
    az account show --query user.name -o tsv
    ```
* Ensure all resources are created in:
  * East US
* Network restrictions or private endpoints are NOT required

***

### Portal Access

* URL: [https://portal.azure.com](https://portal.azure.com)
* Username: [kk\_lab\_user\_main-00dc34062b784e4b@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-00dc34062b784e4b@azurefreekmlprod.onmicrosoft.com)
* Password: \*\*\*

***

## 🔐 Azure Key Vault Encryption Lab (Nautilus DevOps)

### 📌 Objective

The goal of this lab is to secure sensitive data using Azure Key Vault by:

* Creating a Key Vault in Azure
* Creating an RSA encryption key (4096-bit)
* Encrypting a sensitive file using RSA-OAEP
* Decrypting the file and validating integrity

***

## 🖥️ Prerequisites

* Azure CLI installed
* Access to Azure subscription
*   File available:

    ```
    /root/SensitiveData.txt
    ```

***

## 🌐 GUI Method (Azure Portal)

If doing this via Azure Portal:

### Step 1: Create Key Vault

1. Go to: [https://portal.azure.com](https://portal.azure.com)
2. Search **Key Vault**
3. Click **Create**

#### Configuration:

* Subscription: Provided lab subscription
* Resource Group: `kml_rg_main-00dc34062b784e4b`
* Key Vault Name: `nautilus-3581`
* Region: `East US`
* Pricing Tier: Standard
* Soft Delete: Enabled (7 days)
* Access Configuration:
  * Choose **Vault access policy**

Click **Review + Create → Create**

***

### Step 2: Create Key

Inside Key Vault:

1. Go to **Keys**
2. Click **Generate/Import**
3. Configure:
   * Name: `nautilus-key`
   * Key type: RSA
   * Size: 4096
4. Click **Create**

***

### Step 3: Set Access Policy

1. Go to **Access Policies**
2. Click **Add Access Policy**
3. Select permissions:
   * Get
   * List
   * Encrypt
   * Decrypt
4. Select your user/service principal
5. Click **Save**

***

### Step 4: Encryption/Decryption

Done using Azure CLI (below)

***

## 💻 CLI Implementation

### 📍 1. Resource Group Check

```bash
az group list -o table
```

#### Output:

```
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-00dc34062b784e4b  eastus      Succeeded
```

***

### 📍 2. Variables Setup

```bash
RG="kml_rg_main-00dc34062b784e4b"
KV="nautilus-3581"
LOCATION="eastus"
```

***

### 📍 3. Create Key Vault

```bash
az keyvault create \
  --name $KV \
  --resource-group $RG \
  --location $LOCATION \
  --sku standard \
  --retention-days 7 \
  --enable-rbac-authorization false
```

#### Output (truncated):

```
"name": "nautilus-3581",
"location": "eastus",
"enableSoftDelete": true,
"softDeleteRetentionInDays": 7,
"provisioningState": "Succeeded"
```

***

### 📍 4. Get Identity

```bash
az account show --query user.name -o tsv
```

#### Output:

```
0cd95a34-b178-4a95-9db0-962aa0175534
```

***

### 📍 5. Set Access Policy

```bash
PRINCIPAL_ID="0cd95a34-b178-4a95-9db0-962aa0175534"
```

```bash
az keyvault set-policy \
  --name nautilus-3581 \
  --object-id "$PRINCIPAL_ID" \
  --key-permissions get list encrypt decrypt
```

#### Output:

```
"keys": [
  "encrypt",
  "decrypt",
  "get",
  "list"
]
```

***

### 📍 6. Create RSA Key (4096-bit)

```bash
az keyvault key create \
  --vault-name nautilus-3581 \
  --name nautilus-key \
  --kty RSA \
  --size 4096
```

#### Output:

```
"keyOps": ["encrypt","decrypt","sign","verify","wrapKey","unwrapKey"],
"kid": "https://nautilus-3581.vault.azure.net/keys/nautilus-key/..."
```

***

### 📍 7. Base64 Encode File

```bash
base64 /root/SensitiveData.txt > /root/plain.b64
PLAINTEXT_B64=$(tr -d '\n' < /root/plain.b64)
```

***

### 📍 8. Encrypt Data (RSA-OAEP)

```bash
az keyvault key encrypt \
  --vault-name nautilus-3581 \
  --name nautilus-key \
  --algorithm RSA-OAEP \
  --value "$PLAINTEXT_B64" \
  -o json > /root/encrypt.json
```

#### Output (warning only):

```
WARNING: This command is in preview
```

***

### 📍 9. Save Encrypted File

```bash
jq -r '.result' /root/encrypt.json > /root/EncryptedData.bin
```

***

### 📍 10. Decrypt Data

```bash
CIPHERTEXT=$(cat /root/EncryptedData.bin)
```

```bash
az keyvault key decrypt \
  --vault-name nautilus-3581 \
  --name nautilus-key \
  --algorithm RSA-OAEP \
  --value "$CIPHERTEXT" \
  -o json > /root/decrypt.json
```

***

### 📍 11. Decode Base64

```bash
jq -r '.result' /root/decrypt.json > /root/decrypted.b64
base64 -d /root/decrypted.b64 > /root/DecryptedData.txt
```

***

### 📍 12. Verification

```bash
diff /root/SensitiveData.txt /root/DecryptedData.txt
```

#### Output:

```
(no output)
```

✔ Files match successfully

***

## 🎯 Final Result

You successfully:

* Created Azure Key Vault
* Configured access policies
* Generated RSA 4096-bit key
* Performed encryption using RSA-OAEP
* Decrypted and validated data integrity

***

## 🧠 Key Learning

* Key Vault handles cryptographic operations securely
* RSA is used for encrypting small data or keys (not large files directly)
* Base64 encoding is required because Key Vault API accepts text payloads
* Access policies control encryption/decryption permissions

***

<figure><img src=".gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

