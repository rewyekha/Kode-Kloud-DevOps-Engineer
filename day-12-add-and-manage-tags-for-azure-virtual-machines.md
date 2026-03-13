# Day 12: Add and Manage Tags for Azure Virtual Machines

The Nautilus DevOps team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is not tagged properly so they decided to tag it as needed.

Add the tag `Environment=dev` to the virtual machine named `datacenter-vm`.

Use the below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-d41568f8857d4f9d@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-d41568f8857d4f9d@azurefreekmlprod.onmicrosoft.com) |
| Password   | hktGTa7E                                                                                                                                           |

<pre><code><strong>~ ➜  az vm update \
</strong><strong>  --resource-group &#x3C;RESOURCE_GROUP_NAME> \
</strong><strong>  --name datacenter-vm \
</strong><strong>  --set tags.Environment=dev
</strong>-bash: RESOURCE_GROUP_NAME: No such file or directory

<strong>~ ✖ az vm list --output table
</strong>Name           ResourceGroup                 Location    Zones
-------------  ----------------------------  ----------  -------
datacenter-vm  KML_RG_MAIN-D41568F8857D4F9D  westus

<strong>~ ➜  az vm update   --resource-group KML_RG_MAIN-D41568F8857D4F9D  --name datacenter-vm   --set tags.Environment=dev
</strong>{
  "additionalCapabilities": null,
  "applicationProfile": null,
  "availabilitySet": null,
  "billingProfile": null,
  "capacityReservation": null,
  "diagnosticsProfile": null,
  "etag": "\"2\"",
  "evictionPolicy": null,
  "extendedLocation": null,
  "extensionsTimeBudget": null,
  "hardwareProfile": {
    "vmSize": "Standard_B1s",
    "vmSizeProperties": null
  },
  "host": null,
  "hostGroup": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/KML_RG_MAIN-D41568F8857D4F9D/providers/Microsoft.Compute/virtualMachines/datacenter-vm",
  "identity": null,
  "instanceView": null,
  "licenseType": null,
  "location": "westus",
  "managedBy": null,
  "name": "datacenter-vm",
  "networkProfile": {
    "networkApiVersion": null,
    "networkInterfaceConfigurations": null,
    "networkInterfaces": [
      {
        "deleteOption": null,
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-d41568f8857d4f9d/providers/Microsoft.Network/networkInterfaces/datacenter-vmVMNic",
        "primary": null,
        "resourceGroup": "kml_rg_main-d41568f8857d4f9d"
      }
    ]
  },
  "osProfile": {
    "adminPassword": null,
    "adminUsername": "azureuser",
    "allowExtensionOperations": true,
    "computerName": "datacenter-vm",
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
            "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKna6o9PPJ4iiAnmzjU6wgMdV7vHZiRQxbD3wJIwYUVVwvOyrpb5D8KAlRHVUwnQ9uVsiZiWg3U1XNhOzGtamoZdQN88ix0V51rCxP2uUEyGMq6G+Zr1PS534ZlTSmfghjy/WBv500AcUXF/ut4dPzLMx+BAuqi3M0xWQO16iW7BcqXb2CxGH406rFE0p7y2IHYk5MnLVkUWhNgNT+5FNjIJgtCWMTuNUw0Ebn5iY7K3ploKmgeMc19Z+evdz2x3WADMmOO7Pz0SUSesz5O98URCUcKGPfO9UqAFxm44Uink35e+HgGOL6ZMRY0LAxp1bNyXMnnNyHYqiQyTg7o2sT root@azure-client\n",
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
  "resourceGroup": "KML_RG_MAIN-D41568F8857D4F9D",
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
      "diskSizeGb": 128,
      "encryptionSettings": null,
      "image": null,
      "managedDisk": {
        "diskEncryptionSet": null,
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-d41568f8857d4f9d/providers/Microsoft.Compute/disks/datacenter-vm_disk1_2a3a0a3d29594ba288ab4413aa8f4d7a",
        "resourceGroup": "kml_rg_main-d41568f8857d4f9d",
        "securityProfile": null,
        "storageAccountType": "Standard_LRS"
      },
      "name": "datacenter-vm_disk1_2a3a0a3d29594ba288ab4413aa8f4d7a",
      "osType": "Linux",
      "vhd": null,
      "writeAcceleratorEnabled": null
    }
  },
  "tags": {
    "Environment": "dev"
  },
  "timeCreated": "2025-12-27T05:56:32.884972+00:00",
  "type": "Microsoft.Compute/virtualMachines",
  "userData": null,
  "virtualMachineScaleSet": null,
  "vmId": "5412715d-4707-4f96-9ea4-5bcfc43b54c2",
  "zones": null
}

<strong>~ ➜  az vm show \
</strong><strong>  --resource-group KML_RG_MAIN-D41568F8857D4F9D \
</strong><strong>  --name datacenter-vm \
</strong><strong>  --query tags
</strong>{
  "Environment": "dev"
}

~ ➜  
</code></pre>



In Azure, a VM is uniquely identified by **both**:

* **VM name**
* **Resource group name**

You’ve provided the VM name (`datacenter-vm`), but not the **resource group** it belongs to.\
That’s needed whether you use the Azure Portal or Azure CLI.

***

### Option 1: Find the resource group (Azure CLI – recommended)

1. On the **azure-client** host, log in:

```bash
az login
```

2. List VMs to find the resource group:

```bash
az vm list --output table
```

Look for `datacenter-vm` and note its **ResourceGroup** value.

***

### Option 2: Add the tag using Azure CLI

Once you know the resource group, run:

```bash
az vm update \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name datacenter-vm \
  --set tags.Environment=dev
```

✅ This will add the tag:

```
Environment=dev
```

***

### Option 3: Verify the tag

```bash
az vm show \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name datacenter-vm \
  --query tags
```

***

### Option 4: Azure Portal (GUI)

1. Go to [**https://portal.azure.com**](https://portal.azure.com)
2. Navigate to **Virtual Machines**
3. Select **datacenter-vm**
4. Click **Tags**
5. Add:
   * **Name:** Environment
   * **Value:** dev
6. Click **Save**
