---
title: Azure PowerShell Tools 0.8.10 is available
author: Ravikanth C
type: post
date: 2014-10-29T02:05:26+00:00
url: /2014/10/28/azure-powershell-tools-0-8-10-is-available/
categories:
  - Azure
  - News
tags:
  - Azure
  - News
---
If you have missed it, [Scott Gu][1] announced major set of updates to Azure services. At a high level, these changes include,

  * **Marketplace**: Announcing Azure Marketplace and partnerships with key technology partners
  * **Networking**: Network Security Groups, Multi-NIC, Forced Tunneling, Source IP Affinity, and much more
  * **Batch Computing**: Public Preview of the new Azure Batch Computing Service
  * **Automation**: General Availability of the Azure Automation Service
  * **Anti-malware**: General Availability of Microsoft Anti-malware for Virtual Machines and Cloud Services
  * **Virtual Machines**: General Availability of many more VM extensions – PowerShell DSC, Octopus, VS Release Management

This, of course, needs Azure PowerShell Tools update too! Version 0.8.10 of the [Azure cmdlets is now available][2]. This includes new cmdlets to manage Data Factory, Batch Computing, Multi-NIC configurations and so on.

- Azure Data Factory cmdlets in AzureResourceManager mode
  - New-AzureDataFactory
  - New-AzureDataFactoryGateway
  - New-AzureDataFactoryGatewayKey
  - New-AzureDataFactoryHub
  - New-AzureDataFactoryLinkedService
  - New-AzureDataFactoryPipeline
  - New-AzureDataFactoryTable
  - New-AzureDataFactoryEncryptValue
  - Get-AzureDataFactory
  - Get-AzureDataFactoryGateway
  - Get-AzureDataFactoryHub
  - Get-AzureDataFactoryLinkedService
  - Get-AzureDataFactoryPipeline
  - Get-AzureDataFactoryRun
  - Get-AzureDataFactorySlice
  - Get-AzureDataFactoryTable
  - Remove-AzureDataFactory
  - Remove-AzureDataFactoryGateway
  - Remove-AzureDataFactoryHub
  - Remove-AzureDataFactoryLinkedService
  - Remove-AzureDataFactoryPipeline
  - Remove-AzureDataFactoryTable
  - Resume-AzureDataFactoryPipeline
  - Save-AzureDataFactoryLog
  - Set-AzureDataFactoryGateway
  - Set-AzureDataFactoryPipelineActivePeriod
  - Set-AzureDataFactorySliceStatus
  - Suspend-AzureDataFactoryPipeline
- Azure Batch cmdlets in AzureResourceManager mode
  - Set-AzureBatchAccount
  - Remove-AzureBatchAccount
  - New-AzureBatchAccountKey
  - New-AzureBatchAccount
  - Get-AzureBatchAccountKeys
  - Get-AzureBatchAccount
- Azure Network
  - Multi NIC support
    - Add-AzureNetworkInterfaceConfig
    - Get-AzureNetworkInterfaceConfig
    - Remove-AzureNetworkInterfaceConfig
    - Set-AzureNetworkInterfaceConfig
  - Security group support
    - Set-AzureNetworkSecurityGroupToSubnet
    - Set-AzureNetworkSecurityGroupConfig
    - Remove-AzureNetworkSecurityGroupFromSubnet
    - Remove-AzureNetworkSecurityGroupConfig
    - Remove-AzureNetworkSecurityGroup
    - New-AzureNetworkSecurityGroup
    - Get-AzureNetworkSecurityGroupForSubnet
    - Get-AzureNetworkSecurityGroupConfig
    - Get-AzureNetworkSecurityGroup
- Azure Virtual Machine
  - Added Add PublicConfigKey and PrivateConfigKey parameters to SetAzureVMExtension
- Azure Website
  - Set-AzureWebsite exposes new parameters and Get-AzureWebsite returns them
    - SlotStickyConnectionStringNames – connection string names not to be moved during swap operation
    - SlotStickyAppSettingNames – application settings names not to be moved during swap operation
    - AutoSwapSlotName – slot name to swap automatically with after successful deployment

Go ahead and download the new release! As I&#8217;d written last time, [you can use PowerShell to do that][3].

```
#get a list of all MSIs
Get-AzurePowerShellMSI -Passthru

#Download the latest MSI
Get-AzurePowerShellMSI -DownloadLatest -Verbose -Passthru

#Download the latest MSI to a specific destination
Get-AzurePowerShellMSI -DownloadLatest -DownloadPath C:\MyDownloads1 -Verbose -Passthru
```

[1]: https://weblogs.asp.net/scottgu/azure-new-marketplace-network-improvements-new-batch-service-automation-service-more
[2]: https://github.com/Azure/azure-sdk-tools/releases/tag/v0.8.10-October2014
[3]: https://gist.github.com/rchaganti/68a2bb55ed015d19385e