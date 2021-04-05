---
title: Manage your public DNS zones with Azure DNS
author: Jan Egil Ring
type: post
date: 2015-05-11T16:00:40+00:00
url: /2015/05/11/manage-your-public-dns-zones-with-azure-dns/
views:
  - 15463
post_views_count:
  - 2998
categories:
  - Azure
  - Azure DNS
tags:
  - Azure
  - Azure DNS

---
On May 4 2015, Microsoft has released a public preview of a new Microsoft Azure feature – [Azure DNS][1].

Azure DNS is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure. By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools and billing as your other Azure services.

You can find more information in the [Azure DNS documentation][2].

The service is based on Azure Resource Manager (ARM), and can be managed using the Azure PowerShell module. The DNS cmdlets available in the module are added in the 0.9.1 release, which you can download from the [Azure PowerShell repository][3] on GitHub.

You can verify what version you have installed by using `Get-Module –ListAvailable –Name Azure`:

![](/images/adns1.png)

The mode for the module must be switched to AzureResourceManager, and you must add your account and select your subscription as you normally do when working with the Azure PowerShell module:

```powershell
Switch-AzureMode -Name AzureResourceManager
Add-AzureAccount
Get-AzureSubscription
Select-AzureSubscription -SubscriptionName "your subscription name"
```


It’s not currently possible to sign up for the DNS Service preview from the Azure portals, but this can be accomplished by using cmdlets in the Azure PowerShell module.

```powershell
Register-AzureProvider -ProviderNamespace Microsoft.Network -Force
Register-AzureProviderFeature -ProviderNamespace Microsoft.Network -FeatureName azurednspreview -Force
Get-AzureProviderFeature -ProviderNamespace Microsoft.Network -FeatureName azurednspreview
```


It may take up to 24 hours for the registration state to change to Registered:

![](/images/adns2.png)

Before creating a new DNS zone, you must decide what resource group to place the zone into. You may use an existing resource group, or you can create a new one:

```powershell
New-AzureResourceGroup -Name Networking -Location 'West Europe'
```


For testing purposes, I’ve decided to use a subdomain of my powershell.no domain:

```powershell
New-AzureDnsZone -Name azure.powershell.no -ResourceGroupName Networking
```


When the zone is created, you must add the appropriate Name Server (NS) records to the DNS zone at your registrar/hoster (it\`s not yet available to host your domain in Azure). To find the values for the records you need to configure, you can use the Get-AzureDnsRecordSet cmdlet:

```powershell
Get-AzureDnsRecordSet -ZoneName azure.powershell.no -ResourceGroupName Networking -RecordType NS
```


![](/images/adns3.png)

In the DNS Control Panel at my registrar, it looks like this after I added the NS records for the Azure DNS Service:

![](/images/adns4.png)

Now you can start creating DNS records in the new zone, and they should be resolved by the Azure DNS service.

```powershell
New-AzureDnsRecordSet -Name www -RecordType A -ZoneName azure.powershell.no -Ttl 60 |
Add-AzureDnsRecordConfig -Ipv4Address '1.2.3.4' |
Set-AzureDnsRecordSet
```


You can then verify the new record by using familiar tools such as the Resolve-DnsName cmdlet or the traditional nslookup utility:

![](/images/adns5.png)

To further explore the available cmdlets for managing Azure DNS, you can use Get-Command -Module AzureResourceManager -Name _Dns_

![](/images/adns6.png)

You can also find more information and examples on the [Azure DNS Documentation website][4].

[1]: http://azure.microsoft.com/en-us/services/dns/
[2]: http://azure.microsoft.com/en-us/documentation/articles/dns-overview/
[3]: https://github.com/Azure/azure-powershell/releases
[4]: http://azure.microsoft.com/en-us/documentation/articles/dns-getstarted-create-recordset/