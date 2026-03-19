# Day 23: Automating User Data Configuration Using the CLI

The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.

As a member of the Nautilus DevOps Team, your task is to create a VM using Azure CLI with the following specifications:

Instance Name: The VM must be named `devops-vm`.

Image: Use any available Ubuntu image to create this VM.

Custom Script Extension/User Data: Configure the VM to run a custom script during its launch. This script should:

* Install the Nginx package.
* Start the Nginx service.

Network Security Group (NSG): Ensure that the VM allows HTTP traffic on port `80` from the internet.

Instructions:

1. Use Azure CLI commands to set up the VM in the specified configuration.
2. Ensure the VM is accessible from the internet on port 80.
3. The Nginx service should be running after setup.

Use the Azure CLI commands to complete the task.\
`Notes:`

* Create the resources only in the `East US` region.
* You may use the default resource group or create a new one if needed.

***

## Setting Up a Nautilus Web Server VM on Azure

This document outlines the step-by-step process for creating an Ubuntu virtual machine (VM) on Azure to host a web server using **Nginx**, including troubleshooting failed commands and ensuring the VM is publicly accessible on port 80.

***

### Prerequisites

* Azure CLI installed and logged in.
* Access to a resource group in **East US** (we will use `kml_rg_main-1be6f9291f3249e3`).
* Basic knowledge of Linux and SSH.

***

### 1. Check Existing Resource Groups

```bash
~ ✖ az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-1be6f9291f3249e3  eastus      Succeeded
```

> ✅ We have an existing resource group in East US.

***

### 2. Attempt to Create the VM with Incorrect NSG Rule

```bash
~ ➜ az vm create \
  --resource-group kml_rg_main-1be6f9291f3249e3 \
  --name devops-vm \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data install_nginx.sh \
  --nsg-rule HTTP \
  --size Standard_B1s \
  --storage-sku StandardSSD_LRS \
  --os-disk-size-gb 30 \
  --no-wait \
  --output table
az vm create: 'HTTP' is not a valid value for '--nsg-rule'. Allowed values: RDP, SSH, NONE.
```

> ❌ **Mistake:** `--nsg-rule HTTP` is invalid. Only `SSH`, `RDP`, or `NONE` are allowed.

***

### 3. Attempt to Create the VM with Invalid Image

```bash
~ ✖ az vm create \
  --resource-group kml_rg_main-1be6f9291f3249e3 \
  --name devops-vm \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data install_nginx.sh \
  --size Standard_B1s \
  --storage-sku StandardSSD_LRS \
  --os-disk-size-gb 30 \
  --no-wait \
  --output table
Invalid image "UbuntuLTS". Use a valid image URN, custom image name, custom image id, VHD blob URI, or pick an image from ['CentOS85Gen2', 'Debian11', 'OpenSuseLeap154Gen2', 'RHELRaw8LVMGen2', 'SuseSles15SP5', 'Ubuntu2204', 'Ubuntu2404', 'Ubuntu2404Pro', 'FlatcarLinuxFreeGen2', 'Win2022Datacenter', 'Win2022AzureEditionCore', 'Win2019Datacenter', 'Win2016Datacenter', 'Win2012R2Datacenter', 'Win2012Datacenter'].
```

> ❌ **Mistake:** `UbuntuLTS` is not valid. Correct image is **`Ubuntu2204`**.

***

### 4. Create VM with Correct Image

```bash
~ ✖ az vm create \
  --resource-group kml_rg_main-1be6f9291f3249e3 \
  --name devops-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data install_nginx.sh \
  --size Standard_B1s \
  --storage-sku StandardSSD_LRS \
  --os-disk-size-gb 30 \
  --no-wait \
  --output table
SSH key files '/root/.ssh/id_rsa' and '/root/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage, back up your keys to a safe location.
```

> ✅ VM created successfully with Ubuntu 22.04 LTS.

***

### 5. Open Port 80 for HTTP

```bash
~ ➜ az vm open-port --resource-group kml_rg_main-1be6f9291f3249e3 --name devops-vm --port 80
```

> ✅ HTTP port opened for public access.

***

### 6. Check Public IP Address

```bash
~ ➜ az vm list-ip-addresses --name devops-vm --resource-group kml_rg_main-1be6f9291f3249e3 --output table
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
devops-vm         23.100.24.239        10.0.0.4
```

***

### 7. Test HTTP Access (Before Installing Nginx)

```bash
~ ➜ curl http://23.100.24.239
curl: (7) Failed to connect to 23.100.24.239 port 80: Connection refused
```

> ❌ Connection refused because Nginx is not installed yet.

***

### 8. Verify Nginx Installation via Run Command

```bash
~ ✖ az vm run-command invoke \
  --resource-group kml_rg_main-1be6f9291f3249e3 \
  --name devops-vm \
  --command-id RunShellScript \
  --scripts "systemctl status nginx"
{
  "value": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": "Enable succeeded: \n[stdout]\n\n[stderr]\nUnit nginx.service could not be found.\n",
      "time": null
    }
  ]
}
```

> ❌ Nginx is not yet installed.

***

### 9. Install and Start Nginx on VM

```bash
~ ➜ az vm run-command invoke \
  --resource-group kml_rg_main-1be6f9291f3249e3 \
  --name devops-vm \
  --command-id RunShellScript \
  --scripts "sudo apt-get update && sudo apt-get install -y nginx && sudo systemctl start nginx && sudo systemctl enable nginx"
```

> ✅ Nginx installed and started successfully.

***

### 10. Verify Nginx Service

```bash
~ ➜ az vm run-command invoke \
  --resource-group kml_rg_main-1be6f9291f3249e3 \
  --name devops-vm \
  --command-id RunShellScript \
  --scripts "systemctl status nginx"
{
  "value": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": "Enable succeeded: \n[stdout]\n● nginx.service - A high performance web server and a reverse proxy server\n     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)\n     Active: active (running) since Tue 2026-02-24 06:39:00 UTC; 50s ago\n       Docs: man:nginx(8)\n   Main PID: 2458 (nginx)\n      Tasks: 2 (limit: 1009)\n     Memory: 4.8M\n        CPU: 31ms\n     CGroup: /system.slice/nginx.service\n             ├─2458 \"nginx: master process /usr/sbin/nginx -g daemon on; master_process on;\"\n             └─2461 \"nginx: worker process\"\n\nFeb 24 06:39:00 devops-vm systemd[1]: Started A high performance web server and a reverse proxy server.\n"
    }
  ]
}
```

> ✅ Nginx is running and enabled on boot.

***

### 11. Test HTTP Access (After Installing Nginx)

```bash
~ ➜ curl http://23.100.24.239
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
</body>
</html>
```

> ✅ Nginx welcome page is accessible from the internet. Web server setup is complete.

***

### 12. Lessons Learned

1. **NSG Rule Limitation:** `--nsg-rule` only accepts `SSH`, `RDP`, or `NONE`. Use `az vm open-port` for other ports.
2. **Image Name:** Always use a valid image name/URN (`Ubuntu2204`).
3. **Custom Script/Cloud-Init:** Plain shell scripts may not run automatically; consider using cloud-init YAML for automation.
4. **Verification:** Always check service status and public access after VM creation.

