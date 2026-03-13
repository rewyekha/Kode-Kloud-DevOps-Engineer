# Day 10: Attach Public IP to Azure Virtual Machine

The Nautilus DevOps team has already set up a virtual machine and allocated a public IP address. The final task is to attach this public IP to the VM's network interface card (NIC).

An existing VM named `devops-vm-pip` and a public IP address named `devops-pip` already exist.

* Attach the public IP `devops-pip` to the network interface of the VM `devops-vm-pip`.

Make sure the VM is properly assigned the public IP.

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-545e3fa653d9440c@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-545e3fa653d9440c@azurefreekmlprod.onmicrosoft.com) |
| Password   | HZXH-7+$                                                                                                                                           |

```

~ ➜  az vm nic list \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --vm-name devops-vm-pip \
  --query "[0].id" -o tsv
/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-545e3fa653d9440c/providers/Microsoft.Network/networkInterfaces/devops-vm-pipVMNic

~ ➜  az network nic ip-config update \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --nic-name devops-vm-pip-nic \
  --name ipconfig1 \
  --public-ip-address devops-pip
(ResourceNotFound) The Resource 'Microsoft.Network/networkInterfaces/devops-vm-pip-nic' under resource group 'kml_rg_main-545e3fa653d9440c' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix
Code: ResourceNotFound
Message: The Resource 'Microsoft.Network/networkInterfaces/devops-vm-pip-nic' under resource group 'kml_rg_main-545e3fa653d9440c' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix

~ ✖ az network nic ip-config update \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --nic-name devops-vm-pipVMNic \
  --name ipconfig1 \
  --public-ip-address devops-pip
ResourceNotFoundError

~ ✖ az network nic ip-config list \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --nic-name devops-vm-pipVMNic \
  -o table
Name                   Primary    PrivateIPAddress    PrivateIPAddressVersion    PrivateIPAllocationMethod    ProvisioningState    ResourceGroup
---------------------  ---------  ------------------  -------------------------  ---------------------------  -------------------  ----------------------------
ipconfigdevops-vm-pip  True       10.0.0.4            IPv4                       Dynamic                      Succeeded            kml_rg_main-545e3fa653d9440c

~ ➜  az network nic ip-config update \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --nic-name devops-vm-pipVMNic \
  --name ipconfigdevops-vm-pip \
  --public-ip-address devops-pip
{
  "etag": "W/\"13130c03-ab9d-48a2-8165-cbb41f0f915a\"",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-545e3fa653d9440c/providers/Microsoft.Network/networkInterfaces/devops-vm-pipVMNic/ipConfigurations/ipconfigdevops-vm-pip",
  "name": "ipconfigdevops-vm-pip",
  "primary": true,
  "privateIPAddress": "10.0.0.4",
  "privateIPAddressVersion": "IPv4",
  "privateIPAllocationMethod": "Dynamic",
  "provisioningState": "Succeeded",
  "publicIPAddress": {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-545e3fa653d9440c/providers/Microsoft.Network/publicIPAddresses/devops-pip",
    "resourceGroup": "kml_rg_main-545e3fa653d9440c"
  },
  "resourceGroup": "kml_rg_main-545e3fa653d9440c",
  "subnet": {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-545e3fa653d9440c/providers/Microsoft.Network/virtualNetworks/devops-vm-pipVNET/subnets/devops-vm-pipSubnet",
    "resourceGroup": "kml_rg_main-545e3fa653d9440c"
  },
  "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
}

~ ➜  az vm show \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --name devops-vm-pip \
  --show-details \
  --query publicIps -o tsv
172.185.169.79

~ ➜  
```



### 🔍 What we learned (key detail)

From your command:

```
az network nic ip-config list ...
```

You got:

```
Name: ipconfigdevops-vm-pip
```

👉 **Correct IP configuration name:**\
&#xNAN;**`ipconfigdevops-vm-pip`**

❌ `ipconfig1` does **not** exist on this NIC\
❌ `devops-vm-pip-nic` is not the NIC name

***

### ✅ FINAL FIX (This Will Work)

Run **exactly** this command:

```bash
az network nic ip-config update \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --nic-name devops-vm-pipVMNic \
  --name ipconfigdevops-vm-pip \
  --public-ip-address devops-pip
```

⏳ Wait for it to complete (should return JSON, not an error).

***

### ✅ Verify Public IP Is Assigned

```bash
az vm show \
  --resource-group kml_rg_main-545e3fa653d9440c \
  --name devops-vm-pip \
  --show-details \
  --query publicIps -o tsv
```

You should now see a **public IPv4 address**.

***

### ✅ Portal Confirmation (Optional)

Azure Portal → **devops-vm-pip → Overview**

You should now see:

* **Primary NIC public IP** → ✔ Assigned
* **Networking → Public IP address** → `devops-pip`

***

### 🎯 Task Status

✔ Public IP `devops-pip`\
✔ Attached to NIC `devops-vm-pipVMNic`\
✔ VM `devops-vm-pip` is now publicly reachable

