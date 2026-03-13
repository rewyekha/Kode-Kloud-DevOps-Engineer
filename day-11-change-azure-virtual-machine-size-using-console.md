# Day 11: Change Azure Virtual Machine Size Using Console

The Nautilus DevOps team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is underutilized and has decided to change its size to optimize resource usage.

1\) Change the VM size from `Standard_B1s` to `Standard_B2s` for the virtual machine named `nautilus-vm`.

2\) Ensure the VM is in the `running` state after the size change is complete.

Use the below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-23d83f5aad804f0d@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-23d83f5aad804f0d@azurefreekmlprod.onmicrosoft.com) |
| Password   | @5QFPYMN                                                                                                                                           |
| Start Time | Fri Dec 26 04:07:53 UTC 2025                                                                                                                       |
| End Time   | Fri Dec 26 05:07:53 UTC 2025                                                                                                                       |

\
`Notes:`

* Create the resources only in `East US` region.
* Make sure the VM is in the `Running` state after resizing.



<pre><code>
<strong>~ ➜  az vm deallocate \
</strong><strong>  --resource-group kml_rg_main-23d83f5aad804f0d \
</strong><strong>  --name nautilus-vm
</strong>
<strong>~ ➜  az vm resize \
</strong><strong>  --resource-group kml_rg_main-23d83f5aad804f0d \
</strong><strong>  --name nautilus-vm \
</strong><strong>  --size Standard_B2s
</strong>{
  "additionalCapabilities": null,
  "applicationProfile": null,
  "availabilitySet": null,
  "billingProfile": null,
  "capacityReservation": null,
  "diagnosticsProfile": null,
  "etag": "\"5\"",
  "evictionPolicy": null,
  "extendedLocation": null,
  "extensionsTimeBudget": null,
  "hardwareProfile": {
    "vmSize": "Standard_B2s",
    "vmSizeProperties": null
  },
  "host": null,
  "hostGroup": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-23d83f5aad804f0d/providers/Microsoft.Compute/virtualMachines/nautilus-vm",
  "identity": null,
  "instanceView": null,
  "licenseType": null,
  "location": "westus",
  "managedBy": null,
  "name": "nautilus-vm",
  "networkProfile": {
    "networkApiVersion": null,
    "networkInterfaceConfigurations": null,
    "networkInterfaces": [
      {
        "deleteOption": null,
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-23d83f5aad804f0d/providers/Microsoft.Network/networkInterfaces/nautilus-vmVMNic",
        "primary": null,
        "resourceGroup": "kml_rg_main-23d83f5aad804f0d"
      }
    ]
  },
  "osProfile": {
    "adminPassword": null,
    "adminUsername": "azureuser",
    "allowExtensionOperations": true,
    "computerName": "nautilus-vm",
    "customData": null,
    "linuxConfiguration": {
      "disablePasswordAuthentication": true,
      "enableVmAgentPlatformUpdates": null,
      "patchSettings": {
        "assessmentMode": "ImageDefault",
        "automaticByPlatformSettings": null,
        "patchMode": "ImageDefault"
      },
      "provisionVmAgent": true,
      "ssh": {
        "publicKeys": [
          {
            "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0ghktWuFFzwO6YjUKuFm3hGwsXxLaHZuXe5UYxPDXRMK0x8ds13RlBALDv/JOC5pNjHz6wa3j05wg0iISKCYpMTp1a49x9eYDi5ePWCkW5G7EomlcihtTAT2WWRGaqiK51u5XwhF/oapRjdS/Ja8JaZj2Tcb7VhDdOhz0bvZLlXY0y010sC+r2/W6SrJUx6hIhKp0QvttwH2XYqAYoRjjp7Ob6m6MzyFvoP7/MEai12RYPXAR3dTsCpZyMZl2rQEegdhupC/CmdCbau7SnjjazF8zax+LHAXX+lV8Uce4fkQdELuGA0MbSkWqe3N+MjqEidNXw7iVjgJhfkK04fyh root@azure-client\n",
            "path": "/home/azureuser/.ssh/authorized_keys"
          }
        ]
      }
    },
    "requireGuestProvisionSignal": true,
    "secrets": [],
    "windowsConfiguration": null
  },
  "plan": null,
  "platformFaultDomain": null,
  "priority": null,
  "provisioningState": "Succeeded",
  "proximityPlacementGroup": null,
  "resourceGroup": "kml_rg_main-23d83f5aad804f0d",
  "resources": null,
  "scheduledEventsPolicy": null,
  "scheduledEventsProfile": null,
  "securityProfile": {
    "encryptionAtHost": null,
    "encryptionIdentity": null,
    "proxyAgentSettings": null,
    "securityType": "TrustedLaunch",
    "uefiSettings": {
      "secureBootEnabled": true,
      "vTpmEnabled": true
    }
  },
  "storageProfile": {
    "dataDisks": [],
    "diskControllerType": "SCSI",
    "imageReference": {
      "communityGalleryImageId": null,
      "exactVersion": "22.04.202512181",
      "id": null,
      "offer": "0001-com-ubuntu-server-jammy",
      "publisher": "Canonical",
      "sharedGalleryImageId": null,
      "sku": "22_04-lts-gen2",
      "version": "latest"
    },
    "osDisk": {
      "caching": "ReadWrite",
      "createOption": "FromImage",
      "deleteOption": "Detach",
      "diffDiskSettings": null,
      "diskSizeGb": null,
      "encryptionSettings": null,
      "image": null,
      "managedDisk": {
        "diskEncryptionSet": null,
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-23d83f5aad804f0d/providers/Microsoft.Compute/disks/nautilus-vm_disk1_342177db44c144e2ba8a08013e655489",
        "resourceGroup": "kml_rg_main-23d83f5aad804f0d",
        "securityProfile": null,
        "storageAccountType": null
      },
      "name": "nautilus-vm_disk1_342177db44c144e2ba8a08013e655489",
      "osType": "Linux",
      "vhd": null,
      "writeAcceleratorEnabled": null
    }
  },
  "tags": {},
  "timeCreated": "2025-12-26T04:09:12.230753+00:00",
  "type": "Microsoft.Compute/virtualMachines",
  "userData": null,
  "virtualMachineScaleSet": null,
  "vmId": "f949d9c9-6cbf-4568-a805-e90d0fafc3ff",
  "zones": null
}

<strong>~ ➜  az vm start \
</strong><strong>  --resource-group kml_rg_main-23d83f5aad804f0d \
</strong><strong>  --name nautilus-vm
</strong>
<strong>~ ➜  az vm show \
</strong><strong>  --resource-group kml_rg_main-23d83f5aad804f0d \
</strong><strong>  --name nautilus-vm \
</strong><strong>  --show-details \
</strong><strong>  --query "{VM:name, Size:hardwareProfile.vmSize, PowerState:powerState}" \
</strong><strong>  -o table
</strong>VM           Size          PowerState
-----------  ------------  ------------
nautilus-vm  Standard_B2s  VM running

~ ➜  
</code></pre>

### VM Resize Using Azure CLI

**VM Name:** `nautilus-vm`\
**Resource Group:** `kml_rg_main-23d83f5aad804f0d`\
**Region:** East US

***

#### 1️⃣ Stop (Deallocate) the VM

```bash
az vm deallocate \
  --resource-group kml_rg_main-23d83f5aad804f0d \
  --name nautilus-vm
```

Wait until the command finishes.

***

#### 2️⃣ Resize the VM (B1s → B2s)

```bash
az vm resize \
  --resource-group kml_rg_main-23d83f5aad804f0d \
  --name nautilus-vm \
  --size Standard_B2s
```

***

#### 3️⃣ Start the VM

```bash
az vm start \
  --resource-group kml_rg_main-23d83f5aad804f0d \
  --name nautilus-vm
```

***

#### 4️⃣ Verify Size and Running State

```bash
az vm show \
  --resource-group kml_rg_main-23d83f5aad804f0d \
  --name nautilus-vm \
  --show-details \
  --query "{VM:name, Size:hardwareProfile.vmSize, PowerState:powerState}" \
  -o table
```

***

#### ✅ Expected Output

```
VM            Size            PowerState
------------ --------------- -------------
nautilus-vm  Standard_B2s     VM running
```

Once you see this, the **KodeCloud task is completed successfully** 🎯
