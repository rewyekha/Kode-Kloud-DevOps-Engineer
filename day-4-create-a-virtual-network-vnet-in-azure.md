# Day 4: Create a Virtual Network (VNet) in Azure

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations.

Create a Virtual Network (VNet) named `devops-vnet` in the `East US` region with any `IPv4` CIDR block.<br>

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-e2b732cae2a54d95@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-e2b732cae2a54d95@azurefreekmlprod.onmicrosoft.com) |
| Password   | \*\*\*\*                                                                                                                                           |
|            |                                                                                                                                                    |

Create the Virtual Network

```
az network vnet create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --location eastus \
  --address-prefix 10.10.0.0/16

```

### Verify VNet Creation

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --output table
```
