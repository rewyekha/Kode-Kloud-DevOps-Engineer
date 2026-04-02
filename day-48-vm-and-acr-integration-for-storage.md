# Day 48: VM and ACR Integration for Storage

## Azure: VM, ACR, and Blob Storage Integration for Python Application

> **Platform:** KodeKloud | **Cloud:** Microsoft Azure **Difficulty:** Intermediate | **Topic:** Azure VM, ACR, Blob Storage, Docker, Python Flask, SSH

***

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
3. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
4. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
5. [Phase 1: ACR, Storage, and Image Setup — azure-client host](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-acr-storage-and-image-setup--azure-client-host)
   * [Step 1: Set Variables](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-set-variables)
   * [Step 2: Create Azure Container Registry](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-create-azure-container-registry)
   * [Step 3: Inspect Application Files](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-inspect-application-files)
   * [Step 4: Build and Push Docker Image](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-build-and-push-docker-image)
   * [Step 5: Create Storage Account](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-create-storage-account)
   * [Step 6: Create Blob Container and Upload config.json](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-create-blob-container-and-upload-configjson)
6. [Phase 2: Create VM via Azure Portal](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-create-vm-via-azure-portal)
7. [Phase 3: Get VM IP and Copy config.json](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-get-vm-ip-and-copy-configjson)
8. [Phase 4: Configure VM — SSH Session](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-4-configure-vm--ssh-session)
   * [Step 7: SSH into VM and Install Docker](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-ssh-into-vm-and-install-docker)
   * [Step 8: Install Azure CLI on VM](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-install-azure-cli-on-vm)
9. [Phase 5: Run Container on VM](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-5-run-container-on-vm)
   * [Step 9: Login to ACR and Run Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-9-login-to-acr-and-run-container)
   * [Step 10: Verify Container and Test Application](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-10-verify-container-and-test-application)
10. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
11. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
12. [Resource Reference](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-reference)

***

### Lab Overview

The Nautilus DevOps team needs to set up an Azure Virtual Machine to interact with an Azure Blob Storage container for storing and retrieving data. A Python Flask application is containerised, pushed to Azure Container Registry, and run on the VM. The app reads configuration data from a `config.json` file stored in Blob Storage and mounted into the container.

***

### Lab Objectives

1. Create a VM named `nautilus-vm` in East US with SSH public key authentication.
2. Create an ACR named `nautilusacr12889`, build and push the Docker image as `nautilus/python-app:latest`.
3. Create storage account `nautilusstor12889`, blob container `nautilus-config`, and upload `config.json`.
4. Install Docker and Azure CLI on the VM, pull the image from ACR, and run it on port 80.
5. Confirm the application is accessible in the browser.

***

### Prerequisites

| Field           | Value                                                                |
| --------------- | -------------------------------------------------------------------- |
| Portal URL      | `https://portal.azure.com`                                           |
| Username        | `kk_lab_user_main-fc1d2baa2c28478f@azurefreekmlprod.onmicrosoft.com` |
| Password        | `+vaY-EVS`                                                           |
| Region          | `East US`                                                            |
| Resource Group  | `kml_rg_main-fc1d2baa2c28478f`                                       |
| ACR Name        | `nautilusacr12889`                                                   |
| Repository      | `nautilus/python-app`                                                |
| Storage Account | `nautilusstor12889`                                                  |
| Blob Container  | `nautilus-config`                                                    |
| VM Name         | `nautilus-vm`                                                        |
| VM Public IP    | `40.121.7.43`                                                        |

***

### Architecture

```bash
azure-client host
      │
      ├── Build Docker image from /root/pyapp/Dockerfile
      │   └── Push to nautilusacr12889.azurecr.io/nautilus/python-app:latest
      │
      ├── Create nautilusstor12889 storage account
      │   └── Upload config.json to nautilus-config container
      │
      └── scp config.json to nautilus-vm:/home/azureuser/
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│              nautilus-vm (Ubuntu 24.04, East US)            │
│              Public IP: 40.121.7.*                         │
│                                                             │
│  Docker container: nautilus-app                             │
│  Image: nautilusacr12889.azurecr.io/nautilus/python-app     │
│  Port: 0.0.0.0:80 -> 80/tcp                                 │
│  Volume: /home/azureuser/config.json:/app/config.json       │
│                                                             │
│  Response: Welcome to KKE Azure Labs: {'key': 'value', ...} │
└─────────────────────────────────────────────────────────────┘
```

***

### Phase 1: ACR, Storage, and Image Setup — azure-client host

#### Step 1: Set Variables

```bash
RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)
LOCATION="eastus"
ACR_NAME="nautilusacr12889"
REPO="nautilus/python-app"
STORAGE_ACCOUNT="nautilusstor12889"
CONTAINER_NAME="nautilus-config"
```

**Terminal Output:**

```bash
~ ➜  RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)

LOCATION="eastus"

ACR_NAME="nautilusacr12889"
REPO="nautilus/python-app"

STORAGE_ACCOUNT="nautilusstor12889"
CONTAINER_NAME="nautilus-config"

~ ➜
```

Resource group resolved to `kml_rg_main-fc1d2baa2c28478f`.

***

#### Step 2: Create Azure Container Registry

```bash
az acr create \
  --name $ACR_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Basic
```

**Terminal Output:**

```bash
~ ➜  az acr create \
  --name $ACR_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Basic
{
  "adminUserEnabled": false,
  "creationDate": "2026-04-01T10:30:37.395380+00:00",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-fc1d2baa2c28478f/providers/Microsoft.ContainerRegistry/registries/nautilusacr12889",
  "location": "eastus",
  "loginServer": "nautilusacr12889.azurecr.io",
  "name": "nautilusacr12889",
  "provisioningState": "Succeeded",
  "resourceGroup": "kml_rg_main-fc1d2baa2c28478f",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "type": "Microsoft.ContainerRegistry/registries"
}
```

ACR `nautilusacr12889` created with `provisioningState: Succeeded` and login server `nautilusacr12889.azurecr.io`.

Login to ACR:

```bash
az acr login --name $ACR_NAME
```

**Terminal Output:**

```
~ ➜  az acr login --name $ACR_NAME
Login Succeeded
```

***

#### Step 3: Inspect Application Files

Review the Python application and Dockerfile before building:

```bash
cd /root/pyapp
ls
cat app.py
cat Dockerfile
```

**Terminal Output:**

```bash
~/pyapp ➜  ls
app.py  Dockerfile

~/pyapp ➜  cat app.py
from flask import Flask
import json

app = Flask(__name__)

@app.route("/")
def home():
    with open("config.json", "r") as f:
        config = json.load(f)
    return f"Welcome to KKE Azure Labs: {config}"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)

~/pyapp ➜  cat Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```

The Flask app reads `config.json` from `/app/config.json` inside the container and returns its contents as an HTTP response. The Dockerfile uses `python:3.9-slim` as the base image and installs Flask.

***

#### Step 4: Build and Push Docker Image

```bash
docker build -t $ACR_NAME.azurecr.io/$REPO:latest .
docker push $ACR_NAME.azurecr.io/$REPO:latest
az acr repository list --name $ACR_NAME --output table
```

**Terminal Output:**

```bash
~/pyapp ➜  docker build -t $ACR_NAME.azurecr.io/$REPO:latest .
[+] Building 13.8s (9/9) FINISHED                                docker:default
 => [internal] load build definition from Dockerfile                     0.3s
 => [internal] load metadata for docker.io/library/python:3.9-slim       1.9s
 => [1/4] FROM docker.io/library/python:3.9-slim@sha256:2d97f691...      5.7s
 => [2/4] WORKDIR /app                                                   0.2s
 => [3/4] COPY app.py /app/                                              0.2s
 => [4/4] RUN pip install flask                                          4.8s
 => exporting to image                                                   0.4s
 => => writing image sha256:db0295a0eba3d2c0ede130514b8b74c7f4762b77...  0.0s
 => => naming to nautilusacr12889.azurecr.io/nautilus/python-app:latest  0.0s

~/pyapp ➜  docker push $ACR_NAME.azurecr.io/$REPO:latest
The push refers to repository [nautilusacr12889.azurecr.io/nautilus/python-app]
4a4a5dae38b6: Pushed
f230026e29b3: Pushed
4cba2c48c48e: Pushed
c8f6b54339a8: Pushed
298992e09a03: Pushed
4f237755fbae: Pushed
d7c97cb6f1fe: Pushed
latest: digest: sha256:f8b7eec40633d7c3fb1983cb24d9503151671164c10df69f78588c389f6f73ec size: 1783

~/pyapp ➜  az acr repository list --name $ACR_NAME --output table
Result
-----------------
nautilus/python-app
```

Image built in `13.8s` and pushed successfully with digest `sha256:f8b7eec4...`. Repository `nautilus/python-app` confirmed in ACR.

***

#### Step 5: Create Storage Account

```bash
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
```

**Terminal Output:**

```bash
~/pyapp ➜  az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
{
  "accessTier": "Hot",
  "allowBlobPublicAccess": false,
  "creationTime": "2026-04-01T10:33:04.671853+00:00",
  "enableHttpsTrafficOnly": true,
  "kind": "StorageV2",
  "location": "eastus",
  "name": "nautilusstor12889",
  "primaryEndpoints": {
    "blob": "https://nautilusstor12889.blob.core.windows.net/",
    "dfs": "https://nautilusstor12889.dfs.core.windows.net/",
    "file": "https://nautilusstor12889.file.core.windows.net/",
    "queue": "https://nautilusstor12889.queue.core.windows.net/",
    "table": "https://nautilusstor12889.table.core.windows.net/",
    "web": "https://nautilusstor12889.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "type": "Microsoft.Storage/storageAccounts"
}
```

Storage account `nautilusstor12889` created with `provisioningState: Succeeded` and blob endpoint `https://nautilusstor12889.blob.core.windows.net/`.

***

#### Step 6: Create Blob Container and Upload config.json

```bash
CONN=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query connectionString -o tsv)

az storage container create \
  --name $CONTAINER_NAME \
  --connection-string "$CONN"

az storage blob upload \
  --container-name $CONTAINER_NAME \
  --name config.json \
  --file ~/config.json \
  --connection-string "$CONN"
```

**Terminal Output:**

```bash
~/pyapp ➜  az storage container create \
  --name $CONTAINER_NAME \
  --connection-string "$CONN"
{
  "created": true
}

~/pyapp ➜  az storage blob upload \
  --container-name $CONTAINER_NAME \
  --name config.json \
  --file ~/config.json \
  --connection-string "$CONN"
Finished[#############################################################]  100.0000%
{
  "client_request_id": "4ce90b92-2db6-11f1-9c69-16d399057629",
  "content_md5": "lCcAkseFhh3oD/AdzgjX8g==",
  "date": "2026-04-01T10:34:01+00:00",
  "etag": "\"0x8DE8FDA315B187C\"",
  "lastModified": "2026-04-01T10:34:02+00:00",
  "request_server_encrypted": true,
  "version": "2022-11-02"
}
```

Container `nautilus-config` created and `config.json` uploaded at `100%` — `lastModified: 2026-04-01T10:34:02`.

***

### Phase 2: Create VM via Azure Portal

The VM `nautilus-vm` was created manually via the Azure Portal with the following configuration:

| Field                 | Value                          |
| --------------------- | ------------------------------ |
| VM name               | `nautilus-vm`                  |
| Region                | **East US**                    |
| Image                 | **Ubuntu Server 24.04 LTS**    |
| Size                  | **Standard\_B1s**              |
| Authentication type   | **SSH public key**             |
| SSH public key source | **Use existing public key**    |
| Username              | `azureuser`                    |
| Inbound ports         | **SSH (22)** and **HTTP (80)** |
| OS disk type          | **Standard HDD**               |

> The SSH public key (`~/.ssh/id_ed25519.pub`) was generated on the azure-client host and pasted into the portal during VM creation. This enables direct passwordless SSH access from the azure-client.

***

### Phase 3: Get VM IP and Copy config.json

Get the VM public IP using the CLI:

```bash
az vm list-ip-addresses -g $RESOURCE_GROUP -o table
```

**Terminal Output:**

```
~/pyapp ➜  az vm list-ip-addresses -g $RESOURCE_GROUP -o table
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
nautilus-vm       40.121.7.43          10.0.0.4
```

VM Public IP: `40.121.7.43`, Private IP: `10.0.0.4`.

Copy `config.json` from azure-client to the VM:

```bash
scp ~/config.json azureuser@40.121.7.43:/home/azureuser/
```

**Terminal Output:**

```
~/pyapp ➜  scp -i ~/.ssh/id_rsa \
  ~/config.json \
  azureuser@40.121.7.43:/home/azureuser/
Warning: Identity file /root/.ssh/id_rsa not accessible: No such file or directory.
config.json                                100%   37     0.3KB/s   00:00
```

The `id_rsa` warning is informational — the SSH connection succeeded using the default key (`id_ed25519`). The `config.json` file (37 bytes) transferred successfully at `100%`.

***

### Phase 4: Configure VM — SSH Session

#### Step 7: SSH into VM and Install Docker

```bash
ssh azureuser@40.121.7.43
```

**Terminal Output:**

```
~ ➜  ssh -i ~/.ssh/id_rsa azureuser@40.121.7.43
Warning: Identity file /root/.ssh/id_rsa not accessible: No such file or directory.
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-1010-azure x86_64)

  System load:  0.43              Processes:             114
  Usage of /:   5.7% of 28.02GB   Users logged in:       0
  Memory usage: 27%               IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

azureuser@nautilus-vm:~$
```

Install Docker:

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker azureuser
```

**Terminal Output (install summary):**

```bash
azureuser@nautilus-vm:~$ sudo apt install -y docker.io
...
Setting up docker.io (28.2.2-0ubuntu1~24.04.1) ...
info: Adding group `docker' (GID 114) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /usr/lib/systemd/system/docker.socket.
...
azureuser@nautilus-vm:~$ sudo systemctl start docker
azureuser@nautilus-vm:~$ sudo systemctl enable docker
azureuser@nautilus-vm:~$ sudo usermod -aG docker azureuser
```

Docker `28.2.2` installed successfully.

***

#### Step 8: Install Azure CLI on VM

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**Terminal Output (install summary):**

```
azureuser@nautilus-vm:~$ curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
...
Get:1 https://packages.microsoft.com/repos/azure-cli noble/main amd64 azure-cli amd64 2.84.0-1~noble [54.2 MB]
Fetched 54.2 MB in 0s (160 MB/s)
...
Setting up azure-cli (2.84.0-1~noble) ...
```

Azure CLI `2.84.0` installed successfully. Exit and reconnect for Docker group to take effect:

```bash
exit
```

```bash
ssh azureuser@40.121.7.43
```

***

### Phase 5: Run Container on VM

#### Step 9: Login to ACR and Run Container

Enable ACR admin user via portal first:

* Portal → **Container registries** → `nautilusacr12889` → **Access keys** → toggle **Admin user ON**

Login to ACR from the VM:

```bash
docker login nautilusacr12889.azurecr.io
```

**Terminal Output:**

```
azureuser@nautilus-vm:~$ docker login nautilusacr12889.azurecr.io
Username: nautilusacr12889
Password:

WARNING! Your credentials are stored unencrypted in '/home/azureuser/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

Confirm `config.json` is present on the VM:

```bash
ls
```

**Terminal Output:**

```
azureuser@nautilus-vm:~$ ls
config.json
```

Run the container with the config file mounted:

```bash
docker run -d \
  -p 80:80 \
  -v /home/azureuser/config.json:/app/config.json \
  --name nautilus-app \
  nautilusacr12889.azurecr.io/nautilus/python-app:latest
```

**Terminal Output:**

```
azureuser@nautilus-vm:~$ docker run -d \
  -p 80:80 \
  -v /home/azureuser/config.json:/app/config.json \
  --name nautilus-app \
  nautilusacr12889.azurecr.io/nautilus/python-app:latest
Unable to find image 'nautilusacr12889.azurecr.io/nautilus/python-app:latest' locally
latest: Pulling from nautilus/python-app
09208cb16939: Pull complete
b3ec39b36ae8: Pull complete
fc7443084902: Pull complete
ea56f685404a: Pull complete
81025af0c2e3: Pull complete
8e261d61b309: Pull complete
b16d4c1bb8aa: Pull complete
Digest: sha256:f8b7eec40633d7c3fb1983cb24d9503151671164c10df69f78588c389f6f73ec
Status: Downloaded newer image for nautilusacr12889.azurecr.io/nautilus/python-app:latest
45b3babe98456655d0c5d81c6542cc401fc5765604ab67cf49362833717fb546
```

7 image layers pulled. Digest `sha256:f8b7eec4...` matches what was pushed from the azure-client — confirming image integrity.

***

#### Step 10: Verify Container and Test Application

```bash
docker ps
```

**Terminal Output:**

```
azureuser@nautilus-vm:~$ docker ps
CONTAINER ID   IMAGE                                                    COMMAND           CREATED         STATUS         PORTS                                 NAMES
45b3babe9845   nautilusacr12889.azurecr.io/nautilus/python-app:latest   "python app.py"   6 seconds ago   Up 6 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp   nautilus-app
```

Container `nautilus-app` is running `Up 6 seconds` with port mapping `0.0.0.0:80->80/tcp`.

Test the application from inside the VM:

```bash
curl http://40.121.7.43
```

**Terminal Output:**

```bash
azureuser@nautilus-vm:~$ curl http://40.121.7.43
Welcome to KKE Azure Labs: {'key': 'value', 'version': 1}
```

The Flask application returned `Welcome to KKE Azure Labs: {'key': 'value', 'version': 1}` — confirming the app is running, reading `config.json` successfully, and responding on port 80.

***

### Lab Complete

| Task                   | Resource                      | Configuration                                               | Status    |
| ---------------------- | ----------------------------- | ----------------------------------------------------------- | --------- |
| ACR                    | `nautilusacr12889`            | Basic, East US                                              | Confirmed |
| Docker image           | `nautilus/python-app:latest`  | Built from `python:3.9-slim`                                | Pushed    |
| Storage account        | `nautilusstor12889`           | Standard\_LRS, East US                                      | Confirmed |
| Blob container         | `nautilus-config`             | Created                                                     | Confirmed |
| `config.json` uploaded | Blob                          | `100%`, `lastModified: 10:34:02`                            | Confirmed |
| VM                     | `nautilus-vm`                 | Ubuntu 24.04, East US, `40.121.7.43`                        | Running   |
| Docker on VM           | `28.2.2`                      | Installed and enabled                                       | Confirmed |
| Azure CLI on VM        | `2.84.0`                      | Installed                                                   | Confirmed |
| `config.json` on VM    | `/home/azureuser/config.json` | Copied via scp                                              | Confirmed |
| Container              | `nautilus-app`                | Port `80:80`, volume mounted                                | Up        |
| Application response   | `http://40.121.7.43`          | `Welcome to KKE Azure Labs: {'key': 'value', 'version': 1}` | Confirmed |

***

### Key Concepts

#### How the Python App Uses config.json

The Flask application reads `config.json` at runtime on every request:

```python
@app.route("/")
def home():
    with open("config.json", "r") as f:
        config = json.load(f)
    return f"Welcome to KKE Azure Labs: {config}"
```

The file is mounted into the container at `/app/config.json` using the Docker volume flag:

```
-v /home/azureuser/config.json:/app/config.json
```

This means the config file lives on the VM's filesystem but appears inside the container at the path the app expects. Changing `config.json` on the host updates it inside the container instantly without restarting.

#### SSH Key Warning — id\_rsa vs id\_ed25519

The terminal showed this warning multiple times:

```
Warning: Identity file /root/.ssh/id_rsa not accessible: No such file or directory.
```

This appeared because the `ssh` command explicitly referenced `-i ~/.ssh/id_rsa` which did not exist. However, the connection still succeeded because SSH automatically fell back to `~/.ssh/id_ed25519` which was generated and registered with the VM during creation. The warning is harmless but indicates the `-i` flag should reference the correct key file.

#### ACR Admin User for Docker Login

By default, ACR creates registries with admin user disabled. To allow `docker login` from the VM using username/password, the admin user must be enabled:

| Method | Command                                               |
| ------ | ----------------------------------------------------- |
| Portal | ACR → Access keys → toggle Admin user                 |
| CLI    | `az acr update --name $ACR_NAME --admin-enabled true` |

Once enabled, the registry name becomes the username and the admin password from the Access keys page is used for authentication.

#### Two-Terminal Workflow

This lab required careful separation of two contexts:

| Terminal                    | Host           | Purpose                                                    |
| --------------------------- | -------------- | ---------------------------------------------------------- |
| Terminal 1 (azure-client)   | `azure-client` | ACR creation, image build/push, storage setup, `scp` to VM |
| Terminal 2 (VM SSH session) | `nautilus-vm`  | Docker/Azure CLI install, `docker login`, `docker run`     |

The critical mistake to avoid is running `docker run` on the azure-client instead of inside the VM — this causes the container to attempt to connect to the wrong Docker daemon, producing `dial unix... invalid argument` errors.

***

### Resource Reference

| Resource          | Type               | Value                                                                     |
| ----------------- | ------------------ | ------------------------------------------------------------------------- |
| Resource Group    | Azure RG           | `kml_rg_main-fc1d2baa2c28478f`                                            |
| ACR               | Container Registry | `nautilusacr12889`                                                        |
| ACR Login Server  | DNS                | `nautilusacr12889.azurecr.io`                                             |
| Image Repository  | ACR                | `nautilus/python-app`                                                     |
| Image Tag         | Docker             | `latest`                                                                  |
| Image Digest      | SHA256             | `sha256:f8b7eec40633d7c3fb1983cb24d9503151671164c10df69f78588c389f6f73ec` |
| Storage Account   | Microsoft.Storage  | `nautilusstor12889`                                                       |
| Blob Container    | Container          | `nautilus-config`                                                         |
| Blob File         | Blob               | `config.json` (37 bytes)                                                  |
| VM Name           | Microsoft.Compute  | `nautilus-vm`                                                             |
| VM OS             | Ubuntu             | `24.04.4 LTS`                                                             |
| VM Public IP      | Network            | `40.121.7.43`                                                             |
| VM Private IP     | Network            | `10.0.0.4`                                                                |
| Container Name    | Docker             | `nautilus-app`                                                            |
| Docker Version    | VM                 | `28.2.2`                                                                  |
| Azure CLI Version | VM                 | `2.84.0`                                                                  |

***

_Lab completed on 2026-04-01 | Azure Region: East US | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>
