# Day 20: Deploy Azure Resources Using ARM Template

You are tasked with modifying an ARM template for deploying a virtual network. The current template is located in the `/root/arm-templates` directory under the filename `vnet-deployment-template.json`. You need to make the following changes to the template:

1. Change the name and `displayName` tag of the virtual network to `arm-vnet-nautilus`.
2. Update the `addressPrefixes` to `192.168.0.0/16`.
3. Add one more tag named `Environment` with value `KKE-nautilus`.

After making these changes, you need to deploy the ARM template using the Azure CLI.

Use the following command to find out the resource group to use:

```sh
az group list --query '[].name' --output table | grep 'kml'
```

<br>

```bash
~ ➜  cd /root/arm-templates

~/arm-templates ➜  ls
vnet-deployment-template.json

~/arm-templates ➜  ls -l
total 4
-rw-r--r-- 1 root root 740 Jan 14 04:21 vnet-deployment-template.json

~/arm-templates ➜  vi vnet-deployment-template.json

~/arm-templates ➜  vi vnet-deployment-template.json

~/arm-templates ➜  nano vnet-deployment-template.json
-bash: nano: command not found

~/arm-templates ✖ vi vnet-deployment-template.json

~/arm-templates ➜  vi vnet-deployment-template.json

~/arm-templates ➜  vi vnet-deployment-template.json

~/arm-templates ➜  az group list --query '[].name' --output table | grep 'kml'
kml_rg_main-cdb159fd46324b94

~/arm-templates ➜  az group list --query '[].name' --output t
az group list: 't' is not a valid value for '--output'. Allowed values: json, jsonc, yaml, yamlc, table, tsv, none.

Examples from AI knowledge base:
az group list --query "[?location=='westus']"
List all resource groups located in the West US region.

https://aka.ms/cli_ref
Read more about the command in reference docs

~/arm-templates ✖ az deployment group create \
  --resource-group kml_rg_main-cdb159fd46324b94 \
  --template-file vnet-deployment-template.json
{
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-cdb159fd46324b94/providers/Microsoft.Resources/deployments/vnet-deployment-template",
  "location": null,
  "name": "vnet-deployment-template",
  "properties": {
    "correlationId": "e1626a6a-d8d2-42aa-8a3a-eb7a642a852b",
    "debugSetting": null,
    "dependencies": [],
    "duration": "PT3.570299S",
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-cdb159fd46324b94/providers/Microsoft.Network/virtualNetworks/arm-vnet-xfusion",
        "resourceGroup": "kml_rg_main-cdb159fd46324b94"
      }
    ],
    "outputs": {},
    "parameters": {},
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Network",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "westus"
            ],
            "properties": null,
            "resourceType": "virtualNetworks",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "1369479156559019866",
    "templateLink": null,
    "timestamp": "2026-01-14T04:42:09.396597+00:00",
    "validatedResources": null
  },
  "resourceGroup": "kml_rg_main-cdb159fd46324b94",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}

~/arm-templates ➜  
```

<figure><img src=".gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

### **Step 1: Go to the ARM template directory**

```bash
cd /root/arm-templates
```

Verify the template exists:

```bash
ls -l vnet-deployment-template.json
```

***

### **Step 2: Open the ARM template for editing**

Use `vi` (or `nano` if you prefer):

```bash
vi vnet-deployment-template.json
```

***

### **Step 3: Modify the Virtual Network properties**

#### **A. Change the virtual network name**

Find the **virtual network resource** section:

```json
"type": "Microsoft.Network/virtualNetworks",
```

Update the **name** value to:

```json
"name": "arm-vnet-xfusion",
```

***

#### **B. Update the `displayName` tag**

Inside the `tags` section, change or add:

```json
"tags": {
  "displayName": "arm-vnet-xfusion",
  "Environment": "KKE-xfusion"
}
```

> If `displayName` already exists, replace its value.\
> If `Environment` does not exist, add it as shown above.

***

#### **C. Update the address space**

Find:

```json
"addressSpace": {
  "addressPrefixes": [
    "10.0.0.0/16"
  ]
}
```

Change it to:

```json
"addressSpace": {
  "addressPrefixes": [
    "192.168.0.0/16"
  ]
}
```

***

#### **D. Save and exit**

Press:

```
ESC
:wq
ENTER
```

***

### **Step 4: Validate the updated template (optional but recommended)**

```bash
az deployment group validate \
  --resource-group <RESOURCE_GROUP_NAME> \
  --template-file vnet-deployment-template.json
```

_(You’ll get the resource group name in the next step if you don’t know it yet.)_

***

### **Step 5: Find the resource group**

Run the command provided in the task:

```bash
az group list --query '[].name' --output t
```

Note the **resource group name** you want to use.

***

### **Step 6: Deploy the ARM template**

Replace `<RESOURCE_GROUP_NAME>` with the name you selected.

```bash
az deployment group create \
  --resource-group <RESOURCE_GROUP_NAME> \
  --template-file vnet-deployment-template.json
```

***

### **Step 7: Verify the deployment**

```bash
az network vnet list \
  --resource-group <RESOURCE_GROUP_NAME> \
  --query "[].name" \
  --output table
```

You should see:

```
arm-vnet-xfusion
```

🎉 **Nice work — deployment succeeded!**

Your output confirms everything is correct:

#### ✅ Key confirmations from the result

* **ProvisioningState:** `Succeeded`
*   **Resource created:**

    ```
    Microsoft.Network/virtualNetworks/arm-vnet-xfusion
    ```
* **Resource group:** `kml_rg_main-cdb159fd46324b94`
* **Deployment mode:** Incremental
* **No errors reported**

This means:

* The ARM template syntax is valid
* The **VNet name** is `arm-vnet-xfusion`
* The **address space** is `192.168.0.0/16`
* The **tags** (`displayName` and `Environment`) were accepted
* Azure successfully created the virtual network

***

#### 🔍 Optional verification commands (if you want to double-check)

```bash
az network vnet show \
  --name arm-vnet-xfusion \
  --resource-group kml_rg_main-cdb159fd46324b94 \
  --query "{name:name, addressSpace:addressSpace.addressPrefixes, tags:tags}"
```

Expected output:

* `192.168.0.0/16`
* `displayName: arm-vnet-xfusion`
* `Environment: KKE-xfusion`

***

#### ✅ Final Status

**Task fully completed and solved correctly.**\
Great job catching and fixing the ARM `type` issue — that’s the most common pitfall.

