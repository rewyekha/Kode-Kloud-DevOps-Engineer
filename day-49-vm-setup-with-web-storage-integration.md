# Day 49: VM Setup with Web Storage Integration

The Nautilus DevOps team is tasked with setting up an environment to host a static web application. The application will serve static content from an Azure Storage Account, and a Virtual Machine (VM) will be configured to fetch and display this content using Nginx. The Azure Storage Account is used as a secure, centralized location for storing the `index.html` file. The team intentionally keeps this file outside the main source code repository, since that repository contains additional internal application code that should not be exposed to or accessed by the VM. By placing only the required static file in the Storage Account, the team can distribute this asset safely and independently of the full codebase.

The VM should securely download the `index.html` blob directly from the designated container (e.g., using Azure CLI, SAS URL, or REST API) and place it in Nginx’s web root directory so that it is served locally by Nginx. The Storage Account is not mounted, and the Static Website feature is not used. The VM retrieves the file during deployment and may re-fetch it whenever updates are needed. The resources must follow best practices for security, performance, and accessibility.

#### Task Details: <a href="#task-details" id="task-details"></a>

1\) **Create a Virtual Network (VNet) and Subnet**:

* Create a VNet named `nautilus-vnet` in the **East US** region.
* Create a subnet named `nautilus-subnet` within the VNet for the VM.

2\) **Create an Azure Storage Account**:

* Create a storage account named `nautilusstor4379` in the **East US** region with **Locally-redundant storage (LRS)**.
* Create a Blob container named `nautilus-container` in the storage account.
* Upload the `index.html` file located at `/root` on the **client host** to the container `nautilus-container`.
* **Ensure the Storage Account is private and not publicly accessible** by disabling public access for the storage account.

3\) **Create a Virtual Machine (VM)**:

* Create a VM named `nautilus-vm` in the **East US** region.
* Use the **nautilus-vnet** and subnet **nautilus-subnet** for the VM.
* Authentication: Use `SSH public key` authentication. (Please select `use existing public key` option, create public-key locally and paste contents of `~/.ssh/id_rsa.pub`)
* Install **Nginx** on the VM.
*   Download the `index.html` file using a command such as:

    `sudo az storage blob download --account-name nautilusstor4379 --account-key xxxxx --container-name nautilus-container --name index.html --file /var/www/html/index.html`
* Ensure Nginx is configured to serve the file from `/var/www/html/index.html`.

4\) **Verify Setup**:

* Verify that the Nginx web server on the **client host** serves the `index.html` file correctly when accessing the VM's public IP address.

<br>

```bash

~ ➜  ls
index.html

~ ➜  az storage blob upload \
  --account-name nautilusstor4379 \
  --container-name nautilus-container \
  --name index.html \
  --file /root/index.html \
  --auth-mode login

The request may be blocked by network rules of storage account. Please check network rule set using 'az storage account show -n accountname --query networkRuleSet'.
If you want to change the default action to apply when no rule matches, please use 'az storage account update'.
                    

~ ✖ az storage account update \
  --name nautilusstor4379 \
  --default-action Allow
{
  "accessTier": "Hot",
  "accountMigrationInProgress": null,
  "allowBlobPublicAccess": false,
  "allowCrossTenantReplication": false,
  "allowSharedKeyAccess": true,
  "allowedCopyScope": null,
  "azureFilesIdentityBasedAuthentication": null,
  "blobRestoreStatus": null,
  "creationTime": "2026-04-02T04:09:34.840621+00:00",
  "customDomain": null,
  "defaultToOAuthAuthentication": false,
  "dnsEndpointType": "Standard",
  "enableExtendedGroups": null,
  "enableHttpsTrafficOnly": true,
  "enableNfsV3": null,
  "encryption": {
    "encryptionIdentity": null,
    "keySource": "Microsoft.Storage",
    "keyVaultProperties": null,
    "requireInfrastructureEncryption": false,
    "services": {
      "blob": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-04-02T04:09:35.207116+00:00"
      },
      "file": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-04-02T04:09:35.207116+00:00"
      },
      "queue": null,
      "table": null
    }
  },
  "extendedLocation": null,
  "failoverInProgress": null,
  "geoReplicationStats": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-ef25ed6a4bba4d55/providers/Microsoft.Storage/storageAccounts/nautilusstor4379",
  "identity": null,
  "immutableStorageWithVersioning": null,
  "isHnsEnabled": null,
  "isLocalUserEnabled": null,
  "isSftpEnabled": null,
  "isSkuConversionBlocked": null,
  "keyCreationTime": {
    "key1": "2026-04-02T04:09:35.198916+00:00",
    "key2": "2026-04-02T04:09:35.198916+00:00"
  },
  "keyPolicy": null,
  "kind": "StorageV2",
  "largeFileSharesState": null,
  "lastGeoFailoverTime": null,
  "location": "eastus",
  "minimumTlsVersion": "TLS1_2",
  "name": "nautilusstor4379",
  "networkRuleSet": {
    "bypass": "AzureServices",
    "defaultAction": "Allow",
    "ipRules": [],
    "ipv6Rules": [],
    "resourceAccessRules": null,
    "virtualNetworkRules": []
  },
  "primaryEndpoints": {
    "blob": "https://nautilusstor4379.blob.core.windows.net/",
    "dfs": "https://nautilusstor4379.dfs.core.windows.net/",
    "file": "https://nautilusstor4379.file.core.windows.net/",
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://nautilusstor4379.queue.core.windows.net/",
    "table": "https://nautilusstor4379.table.core.windows.net/",
    "web": "https://nautilusstor4379.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "privateEndpointConnections": [],
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Disabled",
  "resourceGroup": "kml_rg_main-ef25ed6a4bba4d55",
  "routingPreference": null,
  "sasPolicy": null,
  "secondaryEndpoints": null,
  "secondaryLocation": null,
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": null,
  "storageAccountSkuConversionStatus": null,
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}

~ ➜  az storage blob upload \
  --account-name nautilusstor4379 \
  --container-name nautilus-container \
  --name index.html \
  --file /root/index.html \
  --auth-mode login

The request may be blocked by network rules of storage account. Please check network rule set using 'az storage account show -n accountname --query networkRuleSet'.
If you want to change the default action to apply when no rule matches, please use 'az storage account update'.
                    

~ ✖ az storage account update \
  --name nautilusstor4379 \
  --public-network-access Enabled
{
  "accessTier": "Hot",
  "accountMigrationInProgress": null,
  "allowBlobPublicAccess": false,
  "allowCrossTenantReplication": false,
  "allowSharedKeyAccess": true,
  "allowedCopyScope": null,
  "azureFilesIdentityBasedAuthentication": null,
  "blobRestoreStatus": null,
  "creationTime": "2026-04-02T04:09:34.840621+00:00",
  "customDomain": null,
  "defaultToOAuthAuthentication": false,
  "dnsEndpointType": "Standard",
  "enableExtendedGroups": null,
  "enableHttpsTrafficOnly": true,
  "enableNfsV3": null,
  "encryption": {
    "encryptionIdentity": null,
    "keySource": "Microsoft.Storage",
    "keyVaultProperties": null,
    "requireInfrastructureEncryption": false,
    "services": {
      "blob": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-04-02T04:09:35.207116+00:00"
      },
      "file": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-04-02T04:09:35.207116+00:00"
      },
      "queue": null,
      "table": null
    }
  },
  "extendedLocation": null,
  "failoverInProgress": null,
  "geoReplicationStats": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-ef25ed6a4bba4d55/providers/Microsoft.Storage/storageAccounts/nautilusstor4379",
  "identity": null,
  "immutableStorageWithVersioning": null,
  "isHnsEnabled": null,
  "isLocalUserEnabled": null,
  "isSftpEnabled": null,
  "isSkuConversionBlocked": null,
  "keyCreationTime": {
    "key1": "2026-04-02T04:09:35.198916+00:00",
    "key2": "2026-04-02T04:09:35.198916+00:00"
  },
  "keyPolicy": null,
  "kind": "StorageV2",
  "largeFileSharesState": null,
  "lastGeoFailoverTime": null,
  "location": "eastus",
  "minimumTlsVersion": "TLS1_2",
  "name": "nautilusstor4379",
  "networkRuleSet": {
    "bypass": "AzureServices",
    "defaultAction": "Allow",
    "ipRules": [],
    "ipv6Rules": [],
    "resourceAccessRules": null,
    "virtualNetworkRules": []
  },
  "primaryEndpoints": {
    "blob": "https://nautilusstor4379.blob.core.windows.net/",
    "dfs": "https://nautilusstor4379.dfs.core.windows.net/",
    "file": "https://nautilusstor4379.file.core.windows.net/",
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://nautilusstor4379.queue.core.windows.net/",
    "table": "https://nautilusstor4379.table.core.windows.net/",
    "web": "https://nautilusstor4379.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "privateEndpointConnections": [],
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Enabled",
  "resourceGroup": "kml_rg_main-ef25ed6a4bba4d55",
  "routingPreference": null,
  "sasPolicy": null,
  "secondaryEndpoints": null,
  "secondaryLocation": null,
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": null,
  "storageAccountSkuConversionStatus": null,
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}

~ ➜  az storage blob upload \
  --account-name nautilusstor4379 \
  --container-name nautilus-container \
  --name index.html \
  --file /root/index.html \
  --auth-mode login

The request may be blocked by network rules of storage account. Please check network rule set using 'az storage account show -n accountname --query networkRuleSet'.
If you want to change the default action to apply when no rule matches, please use 'az storage account update'.
                    

~ ✖ az storage account keys list \
  --account-name nautilusstor4379 \
  --query "[0].value" \
  -o tsv
HE3oXxoK4IwDanBBESDDldvPuavmg31legbOQ0q99NaUd63S11udKJ+2tnz0DBfXRPIiPNndHcu7+ASt+h3Lxg==

~ ➜  az storage blob upload \
  --account-name nautilusstor4379 \
  --account-key HE3oXxoK4IwDanBBESDDldvPuavmg31legbOQ0q99NaUd63S11udKJ+2tnz0DBfXRPIiPNndHcu7+ASt+h3Lxg== \                   
  --container-name nautilus-container \
  --name index.html \
  --file /root/index.html
Alive[##############################################Finished[#############################################################]  100.0000%
{
  "client_request_id": "8ac65480-2e4a-11f1-ac72-3e37d0a5a51c",
  "content_md5": "kRwD5S7cGXOjxxIXBxwY/A==",
  "date": "2026-04-02T04:15:11+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DE906E6F5B3A47\"",
  "lastModified": "2026-04-02T04:15:12+00:00",
  "request_id": "ac54e4df-401e-0061-4057-c248fc000000",
  "request_server_encrypted": true,
  "version": "2022-11-02",
  "version_id": null
}

~ ➜  ssh-keygen -t rsa -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:9xbVNO43d1rBkrXTdZdvomaq+taq4aW7Zef32+wl8IQ root@azure-client
The key's randomart image is:
+---[RSA 2048]----+
|               o*|
|              =+B|
|             o.*+|
|             o+ =|
|        S . E..==|
|         . .+= o=|
|      . +..+o + .|
|     . *.oo..  +.|
|      BB+o.. .oo+|
+----[SHA256]-----+

~ ➜  cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1G+VxMik2X2enn1Veh/fRXcj8Tmi8o5ycZgp/f0OzsankaKbzRgYSXO01OT2cSFeQ0WDPMhVQp3UbIVgFTiL7DLe0oVGzg8SeoCN34aCjqbMxAZAZwupay0Xb18wYBb/zby8a/A2QQ9uACudSNE2p72kexxqfzvgzU0UeLeju1t8dqZD0tzeKgmUEb2z7+NP4i/3EAEtx1R8LGSwIw9XoNNP5bh+iu6RjGlIkYQrh8SbX4Iu1D+KMOXyRrpos138yledX0Fgh8ePK7zyrqcQeGi4YS7pk32dkKvUR462WmAOJcLsubLokLCXUvIveLaFeU141Rl2iIAxvsAEFwXQR root@azure-client

~ ➜  az vm create \
  --resource-group kml_rg_main-ef25ed6a4bba4d55 \
  --name nautilus-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --vnet-name nautilus-vnet \
  --subnet nautilus-subnet \
  --public-ip-sku Basic \
  --security-type Standard \
  --os-disk-sku Standard_LRS \
  --no-wait false
unrecognized arguments: --os-disk-sku Standard_LRS false

Examples from AI knowledge base:
https://aka.ms/cli_ref
Read more about the command in reference docs

~ ✖ az vm create \
  --resource-group kml_rg_main-ef25ed6a4bba4d55 \
  --name nautilus-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --vnet-name nautilus-vnet \
  --subnet nautilus-subnet \
  --public-ip-sku Basic \
  --security-type Standard \
  --os-disk-sku Standard_LRS
unrecognized arguments: --os-disk-sku Standard_LRS

Examples from AI knowledge base:
https://aka.ms/cli_ref
Read more about the command in reference docs

~ ✖ ssh azureuser@20.25.17.223
The authenticity of host '20.25.17.223 (20.25.17.223)' can't be established.
ECDSA key fingerprint is SHA256:nbzHbX5XtS8tnedbUGzjwr5FraZqit9n9I6roq2E68o.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.25.17.223' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-1010-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Apr  2 04:22:55 UTC 2026

  System load:  0.44              Processes:             135
  Usage of /:   5.7% of 28.02GB   Users logged in:       0
  Memory usage: 4%                IPv4 address for eth0: 10.0.1.4
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@nautilus-vm:~$ sudo apt update
Hit:1 http://azure.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://azure.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://azure.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:4 http://azure.archive.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://azure.archive.ubuntu.com/ubuntu noble/universe amd64 Packages [15.0 MB]
Get:6 http://azure.archive.ubuntu.com/ubuntu noble/universe Translation-en [5982 kB]
Get:7 http://azure.archive.ubuntu.com/ubuntu noble/universe amd64 Components [3871 kB]
Get:8 http://azure.archive.ubuntu.com/ubuntu noble/universe amd64 c-n-f Metadata [301 kB]
Get:9 http://azure.archive.ubuntu.com/ubuntu noble/multiverse amd64 Packages [269 kB]
Get:10 http://azure.archive.ubuntu.com/ubuntu noble/multiverse Translation-en [118 kB]
Get:11 http://azure.archive.ubuntu.com/ubuntu noble/multiverse amd64 Components [35.0 kB]
Get:12 http://azure.archive.ubuntu.com/ubuntu noble/multiverse amd64 c-n-f Metadata [8328 B]
Get:13 http://azure.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1872 kB]
Get:14 http://azure.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [342 kB]
Get:15 http://azure.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [177 kB]
Get:16 http://azure.archive.ubuntu.com/ubuntu noble-updates/main amd64 c-n-f Metadata [16.9 kB]
Get:17 http://azure.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1665 kB]
Get:18 http://azure.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [323 kB]
Get:19 http://azure.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [387 kB]
Get:20 http://azure.archive.ubuntu.com/ubuntu noble-updates/universe amd64 c-n-f Metadata [34.2 kB]
Get:21 http://azure.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [2894 kB]
Get:22 http://azure.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [671 kB]
Get:23 http://azure.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Components [212 B]
Get:24 http://azure.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Packages [32.1 kB]
Get:25 http://azure.archive.ubuntu.com/ubuntu noble-updates/multiverse Translation-en [7520 B]
Get:26 http://azure.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:27 http://azure.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 c-n-f Metadata [500 B]
Get:28 http://azure.archive.ubuntu.com/ubuntu noble-backports/main amd64 Packages [40.4 kB]
Get:29 http://azure.archive.ubuntu.com/ubuntu noble-backports/main Translation-en [9208 B]
Get:30 http://azure.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [7348 B]
Get:31 http://azure.archive.ubuntu.com/ubuntu noble-backports/main amd64 c-n-f Metadata [368 B]
Get:32 http://azure.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Packages [30.7 kB]
Get:33 http://azure.archive.ubuntu.com/ubuntu noble-backports/universe Translation-en [18.2 kB]
Get:34 http://azure.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [13.2 kB]
Get:35 http://azure.archive.ubuntu.com/ubuntu noble-backports/universe amd64 c-n-f Metadata [1480 B]
Get:36 http://azure.archive.ubuntu.com/ubuntu noble-backports/restricted amd64 Components [216 B]
Get:37 http://azure.archive.ubuntu.com/ubuntu noble-backports/restricted amd64 c-n-f Metadata [116 B]
Get:38 http://azure.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Packages [780 B]
Get:39 http://azure.archive.ubuntu.com/ubuntu noble-backports/multiverse Translation-en [372 B]
Get:40 http://azure.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Components [212 B]
Get:41 http://azure.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 c-n-f Metadata [116 B]
Get:42 http://azure.archive.ubuntu.com/ubuntu noble-security/main amd64 Packages [1551 kB]
Get:43 http://azure.archive.ubuntu.com/ubuntu noble-security/main Translation-en [250 kB]
Get:44 http://azure.archive.ubuntu.com/ubuntu noble-security/main amd64 Components [21.6 kB]
Get:45 http://azure.archive.ubuntu.com/ubuntu noble-security/main amd64 c-n-f Metadata [10.7 kB]
Get:46 http://azure.archive.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1167 kB]
Get:47 http://azure.archive.ubuntu.com/ubuntu noble-security/universe Translation-en [225 kB]
Get:48 http://azure.archive.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]
Get:49 http://azure.archive.ubuntu.com/ubuntu noble-security/universe amd64 c-n-f Metadata [22.8 kB]
Get:50 http://azure.archive.ubuntu.com/ubuntu noble-security/restricted amd64 Components [212 B]
Get:51 http://azure.archive.ubuntu.com/ubuntu noble-security/multiverse amd64 Packages [28.8 kB]
Get:52 http://azure.archive.ubuntu.com/ubuntu noble-security/multiverse Translation-en [6980 B]
Get:53 http://azure.archive.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [212 B]
Get:54 http://azure.archive.ubuntu.com/ubuntu noble-security/multiverse amd64 c-n-f Metadata [396 B]
Fetched 37.9 MB in 13s (2941 kB/s)                                        
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
14 packages can be upgraded. Run 'apt list --upgradable' to see them.
azureuser@nautilus-vm:~$ sudo apt install nginx -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  nginx-common
Suggested packages:
  fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  nginx nginx-common
0 upgraded, 2 newly installed, 0 to remove and 14 not upgraded.
Need to get 565 kB of archives.
After this operation, 1596 kB of additional disk space will be used.
Get:1 http://azure.archive.ubuntu.com/ubuntu noble-updates/main amd64 nginx-common all 1.24.0-2ubuntu7.6 [43.5 kB]
Get:2 http://azure.archive.ubuntu.com/ubuntu noble-updates/main amd64 nginx amd64 1.24.0-2ubuntu7.6 [521 kB]
Fetched 565 kB in 0s (12.6 MB/s)
Preconfiguring packages ...
Selecting previously unselected package nginx-common.
(Reading database ... 69217 files and directories currently installed.)
Preparing to unpack .../nginx-common_1.24.0-2ubuntu7.6_all.deb ...
Unpacking nginx-common (1.24.0-2ubuntu7.6) ...
Selecting previously unselected package nginx.
Preparing to unpack .../nginx_1.24.0-2ubuntu7.6_amd64.deb ...
Unpacking nginx (1.24.0-2ubuntu7.6) ...
Setting up nginx-common (1.24.0-2ubuntu7.6) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
Setting up nginx (1.24.0-2ubuntu7.6) ...
 * Upgrading binary nginx                                           [ OK ] 
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for ufw (0.36.2-6) ...
Scanning processes...                                                      
Scanning linux images...                                                   

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
azureuser@nautilus-vm:~$ sudo systemctl enable nginx
Synchronizing state of nginx.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable nginx
azureuser@nautilus-vm:~$ sudo systemctl start nginx
azureuser@nautilus-vm:~$ curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
Hit:1 http://azure.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://azure.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://azure.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:4 http://azure.archive.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20240203).
ca-certificates set to manually installed.
curl is already the newest version (8.5.0-2ubuntu10.8).
curl set to manually installed.
gnupg is already the newest version (2.4.4-2ubuntu17.4).
gnupg set to manually installed.
lsb-release is already the newest version (12.0-2).
lsb-release set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 14 not upgraded.
Need to get 3970 B of archives.
After this operation, 36.9 kB of additional disk space will be used.
Get:1 http://azure.archive.ubuntu.com/ubuntu noble-updates/universe amd64 apt-transport-https all 2.8.3 [3970 B]
Fetched 3970 B in 0s (150 kB/s)               
Selecting previously unselected package apt-transport-https.
(Reading database ... 69265 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.8.3_all.deb ...
Unpacking apt-transport-https (2.8.3) ...
Setting up apt-transport-https (2.8.3) ...
Scanning processes...                                                      
Scanning linux images...                                                   

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
Types: deb
URIs: https://packages.microsoft.com/repos/azure-cli/
Suites: noble
Components: main
Architectures: amd64
Signed-by: /etc/apt/keyrings/microsoft.gpg
Hit:1 http://azure.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://azure.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://azure.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:4 http://azure.archive.ubuntu.com/ubuntu noble-security InRelease
Get:5 https://packages.microsoft.com/repos/azure-cli noble InRelease [3564 B]
Get:6 https://packages.microsoft.com/repos/azure-cli noble/main amd64 Packages [2072 B]
Fetched 5636 B in 1s (4775 B/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  azure-cli
0 upgraded, 1 newly installed, 0 to remove and 14 not upgraded.
Need to get 54.2 MB of archives.
After this operation, 610 MB of additional disk space will be used.
Get:1 https://packages.microsoft.com/repos/azure-cli noble/main amd64 azure-cli amd64 2.84.0-1~noble [54.2 MB]
Fetched 54.2 MB in 1s (67.1 MB/s)    
Selecting previously unselected package azure-cli.
(Reading database ... 69269 files and directories currently installed.)
Preparing to unpack .../azure-cli_2.84.0-1~noble_amd64.deb ...
Unpacking azure-cli (2.84.0-1~noble) ...
Setting up azure-cli (2.84.0-1~noble) ...
Scanning processes...                                                      
Scanning linux images...                                                   

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
azureuser@nautilus-vm:~$ sudo az storage blob download \
  --account-name nautilusstor4379 \
  --account-key HE3oXxoK4IwDanBBESDDldvPuavmg31legbOQ0q99NaUd63S11udKJ+2tnz0DBfXRPIiPNndHcu7+ASt+h3Lxg== \
  --container-name nautilus-container \
  --name index.html \
  --file /var/www/html/index.html
Alive[################################################################]  10Finished[#############################################################]  100.0000%
{
  "container": "nautilus-container",
  "content": "",
  "contentMd5": null,
  "deleted": false,
  "encryptedMetadata": null,
  "encryptionKeySha256": null,
  "encryptionScope": null,
  "hasLegalHold": null,
  "hasVersionsOnly": null,
  "immutabilityPolicy": {
    "expiryTime": null,
    "policyMode": null
  },
  "isAppendBlobSealed": null,
  "isCurrentVersion": null,
  "lastAccessedOn": null,
  "metadata": {},
  "name": "index.html",
  "objectReplicationDestinationPolicy": null,
  "objectReplicationSourceProperties": [],
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobTier": null,
    "blobTierChangeTime": null,
    "blobTierInferred": null,
    "blobType": "BlockBlob",
    "contentLength": 233,
    "contentRange": "bytes 0-232/233",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "kRwD5S7cGXOjxxIXBxwY/A==",
      "contentType": "text/html"
    },
    "copy": {
      "completionTime": null,
      "destinationSnapshot": null,
      "id": null,
      "incrementalCopy": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "creationTime": "2026-04-02T04:15:12+00:00",
    "deletedTime": null,
    "etag": "\"0x8DE906E6F5B3A47\"",
    "lastModified": "2026-04-02T04:15:12+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "pageBlobSequenceNumber": null,
    "pageRanges": null,
    "rehydrationStatus": null,
    "remainingRetentionDays": null,
    "serverEncrypted": true
  },
  "rehydratePriority": null,
  "requestServerEncrypted": true,
  "snapshot": null,
  "tagCount": null,
  "tags": null,
  "versionId": null
}
azureuser@nautilus-vm:~$ sudo systemctl restart nginx
azureuser@nautilus-vm:~$ 
```

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>
