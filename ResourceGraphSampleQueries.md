## Example Azure Resource Graph queries

## Usage
Login using "Login-AzAccount" and simply copy/paste the query. 

If you prefer to query a single subscription, al you need to do so add the "-subscription" parameter to the Search-AzGraph command
like Search-AzGraph -subscription "<SUBSCRIPTIONID>" -Query "distinct(tenantId), subscriptionId"

## Basic Information
Get delegated Tenants + Subscriptions

```azurepowershell-interactive
Search-AzGraph -Query "distinct(tenantId), subscriptionId"
```

Top 10 resource by type

```azurepowershell-interactive
Search-AzGraph -Query "summarize count() by type 
| project type, total=count_ 
| top 10  by type 
| order by total desc"
```

Resources by location

```azurepowershell-interactive
Search-AzGraph -Query "summarize count() by location 
| project location, total=count_ 
| order by total desc"
```

## Web Apps

Check for HTTPS Only on App Services

```azurepowershell-interactive
Search-AzGraph -Query "extend httpsOnly = aliases['Microsoft.Web/sites/httpsOnly'] 
| where type =~'Microsoft.Web/Sites' and httpsOnly =~ 'false' 
| project AppService=['name'], Kind=['kind'], Subscription=['subscriptionId']"
```

Stopped App Services

```azurepowershell-interactive
Search-AzGraph -Query "extend state = aliases['Microsoft.Web/sites/state'] 
| where type=~'Microsoft.Web/Sites' and state =~ 'stopped' 
| project AppService=['name'], Kind=['kind'], State=['state'], Subscription=['subscriptionId']"
```

App Service Plans count by SKU

```azurepowershell-interactive
Search-AzGraph -Query "extend sku = aliases['Microsoft.Web/serverfarms/sku.name'] 
| where type=~'Microsoft.Web/serverfarms' 
| summarize count() by tostring(sku) 
| project sku, total=count_"
```

App Service Plans Basic Information

```azurepowershell-interactive
Search-AzGraph -Query "extend sku = aliases['Microsoft.Web/serverfarms/sku.name'] 
| extend NumberOfApps = aliases['Microsoft.Web/serverFarms/numberOfSites'] 
| where type=~'Microsoft.Web/serverfarms' 
| project Name=['name'], sku, NumberOfApps, Location=['location']"
```

App Service plans count by Web Apps

```azurepowershell-interactive
Search-AzGraph -Query "extend NumberOfApps = aliases['Microsoft.Web/serverFarms/numberOfSites'] 
|where type=~'Microsoft.Web/serverfarms'
| project Name=['name'], NumberOfApps, Location=['location']"
```

## Virtual machines

VMs overview

```azurepowershell-interactive
Search-AzGraph -Query "extend OS = aliases['Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType'] 
| extend SKU = aliases['Microsoft.Compute/virtualMachines/sku.name'] 
| extend OSSpecific = aliases['Microsoft.Compute/virtualMachines/storageProfile.imageReference.offer'] 
| extend Version = aliases['Microsoft.Compute/virtualMachines/storageProfile.imageReference.sku'] 
| where type =~ 'Microsoft.Compute/VirtualMachines' 
| project name, OS, OSSpecific, Version, SKU, Location=['location'], Subscription=['subscriptionId']" |ft
```

VMs count by operating system

```azurepowershell-interactive
Search-AzGraph -Query "extend Os = aliases['Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType'] 
| where type =~ 'Microsoft.Compute/virtualmachines' 
| summarize count() by tostring(Os) 
| project Os, total=count_"
```

VMs count by image offer

```azurepowershell-interactive
Search-AzGraph -Query "extend OsOffer = aliases['Microsoft.Compute/virtualMachines/storageProfile.imageReference.offer'] 
| where type =~ 'Microsoft.Compute/virtualmachines' 
| summarize count() by tostring(OsOffer) 
| project OsOffer, total=count_"
```

VMs count by size 

```azurepowershell-interactive
Search-AzGraph -Query "extend sku = aliases['Microsoft.Compute/virtualMachines/sku.name'] 
| where type=~'Microsoft.Compute/virtualMachines' 
| summarize count() by tostring(sku) 
| project sku, total=count_"
```

## SQL Databases 

PaaS SQL Databases count by type

```azurepowershell-interactive
Search-AzGraph -Query "where type=~ 'Microsoft.DBforMySQL/servers' 
or type=~'Microsoft.SQL/servers/databases' 
or type=~'Microsoft.DBforPostgreSQL/servers' 
or type=~'Microsoft.DBforMariaDB/servers' 
| summarize count() by type 
| project type, total=count_ 
| order by total desc"
```

PaaS SQL Databases count by location

```azurepowershell-interactive
Search-AzGraph -Query "where type=~ 'Microsoft.DBforMySQL/servers' 
or type=~'Microsoft.SQL/servers/databases' 
or type=~'Microsoft.DBforPostgreSQL/servers' 
or type=~'Microsoft.DBforMariaDB/servers'
| summarize count() by location 
| project location, total=count_ 
| order by total desc"
```

PaaS SQL Databases overview

```azurepowershell-interactive
Search-AzGraph -Query "where type=~ 'Microsoft.DBforMySQL/servers' 
or type=~'Microsoft.SQL/servers/databases' 
or type=~'Microsoft.DBforPostgreSQL/servers' 
or type=~'Microsoft.DBforMariaDB/servers'
| project name, type, location, subscriptionId"
```

## Storage Accounts

Storage Accounts count by Location

```azurepowershell-interactive
Search-AzGraph -Query "where type =~ 'microsoft.storage/storageaccounts' 
| summarize count() by location| project Location=['location'], Total=count_"
```

Storage accounts HTTPS Only check count by Subscription

```azurepowershell-interactive
Search-AzGraph -Query "extend HTTPSOnly = aliases['Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly'] 
| where type =~ 'microsoft.storage/storageaccounts' and HTTPSOnly =~ 'false' 
| summarize count() by subscriptionId 
| project subscriptionId, Total=count_"
```

Storage Accounts overview
```azurepowershell-interactive
Search-AzGraph -Query "extend HTTPSOnly = aliases['Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly'] 
| extend Type = aliases['Microsoft.Storage/storageAccounts/accountType'] 
| extend BlobEncryption = aliases['Microsoft.Storage/storageAccounts/enableBlobEncryption'] 
| extend FileEncryption = aliases['Microsoft.Storage/storageAccounts/enableFileEncryption'] 
| where type =~ 'microsoft.storage/storageaccounts'
| project Name=['name'], Kind=['kind'], Type, HTTPSOnly, BlobEncryption, FileEncryption, Location=['location']" 
|ft
```
