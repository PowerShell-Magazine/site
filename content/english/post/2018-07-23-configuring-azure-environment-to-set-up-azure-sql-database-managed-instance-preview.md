---
title: Configuring Azure environment to set up Azure SQL Database Managed Instance (preview)
author: Jovan Popovic
type: post
date: 2018-07-23T16:00:34+00:00
url: /2018/07/23/configuring-azure-environment-to-set-up-azure-sql-database-managed-instance-preview/
views:
  - 8794
post_views_count:
  - 6678
categories:
  - Azure
  - Azure SQL
tags:
  - Azure
  - Azure SQL

---
[Azure SQL Database Managed Instance (preview)][1] is a new database service in Azure. This is a fully-managed latest version of SQL Server database engine hosted in Azure and managed by Azure and SQL Server teams.

Azure SQL Managed Instance is your private database engine PaaS resource that is placed in your Azure VNet with assigned private IP from the range that you can choose. You need to pre-configure Azure environment where Managed Instance will be placed before creation of new managed instances.

Although you can create and configure all necessary VNets, subnets, and network access rules using the Azure portal, this could be error-prone task because you need to follow [the rules described here][2]. The better approach would be to create PowerShell script that creates and configures your Azure environment where you will place your Managed Instances. In the following sections, you will see one example of the PowerShell code that configures the environment for the Azure SQL Database Managed Instance.

### Configuring environment

In order to provision SQL Database Managed Instances, you would need to do the following three important things:

  1. Create VNet where your Managed Instances will be injected.
  2. Create a dedicated subnet in your VNet where Managed Instances should be placed.
  3. Set user-defined route table that would enable communication between your Managed Instances in private subnet and Azure management service.

In the following sections we describe how to configure these elements using PowerShell. The assumption is that you already have Azure subscription and that you can connect to your Azure account using PowerShell.

### Configuring virtual network

In the first step, you need to create a VNet where your Managed Instances will be placed. In our example, VNet will use IP range 10.0.0.0/16 – you can change this according to your needs.

We also create the default subnet with a range 10.0.0.0/24 where you could place some of the resources that could communicate to Managed Instances (for example, VMs that you would use to install apps that will access managed instance). **Note that other resources cannot be placed in the subnet that is dedicated to Managed Instances.** If you don’t need other resources in the VNet, you can use this subnet to place Managed Instances.

The following commands create new resource group called myPowerShellMagazineResourceGroup&nbsp; and VNet called myPowerShellMagazineNetwork in West Central US region:

```powershell
$resourceGroup = 'myPowerShellMagazineResourceGroup'
$location = 'West Central US'
$vNetName = 'myPowerShellMagazineNetwork'
New-AzureRmResourceGroup -ResourceGroupName $resourceGroup -Location $location
$virtualNetwork = New-AzureRmVirtualNetwork -ResourceGroupName $resourceGroup -Location $location -Name $vNetName -AddressPrefix 10.0.0.0/16
$subnetConfig = Add-AzureRmVirtualNetworkSubnetConfig -Name default -AddressPrefix 10.0.0.0/24 -VirtualNetwork $virtualNetwork
```

Now, the first step is done and you need to create one or more additional subnets that would be dedicated to your Managed Instances.

### Configuring subnet for Managed Instance

Every Managed Instance is placed in a subnet that defines the boundary of IP addresses that every instance can take. Managed Instances don’t have fixed IP addresses. Azure service that controls and manages instances can move an instance to a different IP addresses if OS or database engine code is patched/upgraded, some problem is detected and the instance needs to be moved to the new location, or if you want to assign more resources to the instance so it needs to be re-allocated to a machine with more resources.

Managed Instances will assume that they can take any place/IP address within the IP range of the subnet, so if you accidentally put some VM or other resource in this subnet it will clash with some Managed Instance. You would need to plan carefully how big is IP range that you would assign to this subnet because it cannot be changed once you create first resource in the subnet. IP range depends on the number of instances that you want to place in subnet and you would need at least two IP addresses per each instance and 4 IP addresses reserved for internal services (this might be changed in the future).

The subnet is dedicated to SQL Database Managed Instances. This subnet cannot contain any other resource such as VMs that could take some IP address in the subnet.

Let’s create a new subnet called “mi” with the IP address range 10.0.1.0/24 (this could be changed according to your needs):

```powershell
$subnetConfigMi = Add-AzureRmVirtualNetworkSubnetConfig -Name mi -AddressPrefix 10.0.1.0/24 -VirtualNetwork $virtualNetwork

$virtualNetwork | Set-AzureRmVirtualNetwork
```

If you need more subnets where you want to group and organize your instances, you can repeat this code with different -Name and -AddressPrefix values.

### Enable access to Azure Management Service

The final step is to configure access that would enable the managed instances in the private IP range to communicate with the Azure services that manages them.

Managed Instance subnet must have access to Azure services that gets the heartbeat signals from the Managed Instances that are placed in your subnet. These signals enable the service to check instance health and manage the instance (for example, perform regular backups, failover instance if something is wrong, etc.).

You need to create one route table that will have address prefix 0.0.0.0/0 and next hop set to “Internet” to enable this communication, as shown in the following script:

```powershell
$routeTableMiManagementService = New-AzureRmRouteTable -Name 'myRouteTableMiManagementService' -ResourceGroupName $resourceGroup -location $location

Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $virtualNetwork -Name 'mi' -AddressPrefix 10.0.1.0/24 -RouteTable $routeTableMiManagementService |
Set-AzureRmVirtualNetwork

Get-AzureRmRouteTable -ResourceGroupName $resourceGroup -Name 'myRouteTableMiManagementService' |
Add-AzureRmRouteConfig -Name 'ToManagedInstanceManagementService' -AddressPrefix 0.0.0.0/0 -NextHopType 'Internet' |
Set-AzureRmRouteTable
```

> NOTE: User route table with described configuration is the current requirement during the public preview period. This requirement will change in future and enable you to specify narrow set of IP ranges for the traffic that goes outside the subnet. Always check the Managed Instance documentation to see the latest security rules.

### Conclusion

Once you finish with the steps described in this article, you will have prepared environment where you can create Azure SQL Database Managed Instances.

You can create Managed instances using Azure portal, ARM templates, Azure PowerShell, and Azure CLI. When you use some of these methods, you would need to provide resource group, VNet, and subnet for your managed instances.

Probably is the best to do it first in the Azure portal because you can visually see how to configure the instance, and then you can automate it using Azure PowerShell and following the steps from this article.

[1]: https://docs.microsoft.com/en-us/azure/sql-database/sql-database-managed-instance-transact-sql-information
[2]: https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/03/14/how-to-configure-network-for-azure-sql-managed-instance/