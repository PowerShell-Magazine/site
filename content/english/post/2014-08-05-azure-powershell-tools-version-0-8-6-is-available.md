---
title: Azure PowerShell Tools version 0.8.6 is available
author: Ravikanth C
type: post
date: 2014-08-05T07:00:20+00:00
url: /2014/08/05/azure-powershell-tools-version-0-8-6-is-available/
categories:
  - News
  - Azure
tags:
  - Azure
  - News

---
The [Azure PowerShell Tools][1] are updated very often. But, this [0.8.6 release][2], is special and has many features that I was looking forward to. This release has the new Azure VM DSC extension cmdlets, non-interactive login using _Add-AzureAccount_, a new set of cmdlets for Windows Azure Pack, and so on.

Here is a list of new features and bug fixes in this release:

- Non-interactive login support for Microsoft Organizational account with *`Add-AzureAccount -Credential`*
- Upgrade cloud service cmdlets dependencies to Azure SDK 2.4
- Compute
  - PowerShell DSC VM extension
    - Get-AzureVMDscExtension
    - Remove-AzureVMDscExtension
    - Set-AzureVMDscExtension
    - Publish-AzureVMDscConfiguration
  - Added CompanyName and SupportedOS parameters to Publish-AzurePlatformExtension
  - New-AzureVM will display a warning instead of an error when the service already exists in the same subscription
  - Added Version parameter to generic service extension cmdlets
  - Changed the ShowInGUI parameter to DoNotShowInGUI in Update-AzureVMImage
- SQL Database
  - Added OfflineSecondary parameter to Start-AzureSqlDatabaseCopy
  - Database copy cmdlets will return 2 more properties: IsOfflineSecondary and IsTerminationAllowed
- Windows Azure Pack
  - New-WAPackCloudService
  - Get-WAPackCloudService
  - Remove-WAPackCloudService
  - New-WAPackVMRole
  - Get-WAPackVMRole
  - Set-WAPackVMRole
  - Remove-WAPackVMRole
  - New-WAPackVNet
  - Remove-WAPackVNet
  - New-WAPackVMSubnet
  - Get-WAPackVMSubnet
  - Remove-WAPackVMSubnet
  - New-WAPackStaticIPAddressPool
  - Get-WAPackStaticIPAddressPool
  - Remove-WAPackStaticIPAddressPool
  - Get-WAPackLogicalNetwork

I will write more about the Azure VM DSC extension in an upcoming article.

[1]: https://github.com/Azure/azure-sdk-tools/releases
[2]: https://github.com/Azure/azure-sdk-tools/releases/tag/v0.8.6-August2014