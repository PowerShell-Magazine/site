---
title: Holiday gift from Windows PowerShell team – DSC Resource Kit Wave 9
author: Ravikanth C
type: post
date: 2014-12-18T19:34:27+00:00
url: /2014/12/18/holiday-gift-from-windows-powershell-team-dsc-resource-kit-wave-9/
categories:
  - PowerShell DSC
  - News
tags:
  - News
  - PowerShell DSC

---
Windows PowerShell team just released a [new wave of DSC resources][1] taking the overall DSC resource count to 172! This is a great accomplishment within a single year! Great job.

This [wave contains updates to various existing resources and 5 new DSC resource modules][2].  

| Name               | New,  or Updated | What was added?                                              |
| ------------------ | ---------------- | ------------------------------------------------------------ |
| xAzure             | Updated          | Added a new resource, xAzureVMDscExtension, which  provides the ability to use the DSC Extension for Azure VM’s to apply configurations to new or existing VM’s.Also addressed issues reported in xAzureStorageAccount |
| xAzurePack         | New              | xAzurePack adds 8 new resources used for installation and configuration of Windows Azure Pack. The resources are xAzurePackSetup, xAzurePackUpdate, AzurePackAdmin, xAzurePackFQDN, xAzurePackDatabaseSetting, xAzurePackIdentityProvider, xAzurePackRelyingParty, xAzurePackResourceProvider |
| xExchange          | Updated          | Added 5 new resources: xExchInstall, xExchJetstress, xExchJetstressCleanup, xExchUMCallRouterSettings, xExchWaitForADPrep. Additional improvements were made in xExchActiveSyncVirtualDirectory,  xExchAutoMountPoint, xExchExchangeCertificate, xExchMailboxDatabase, ExchOutlookAnywhere, xExchUMService, and the related samples. |
| xInternetExplorer  | New              | Allows DSC to configure IE home page(s)                      |
| xPsExecutionPolicy | New              | Enables configuring the PS Execution Policy via DSC          |
| xScom              | Updated          | Adds new 5 resources: xSCOMManagementServerUpdate, SCOMWebConsoleServerUpdate, and xSCOMConsoleUpdate for updating OM to Update Rollup 4, plus xSCOMAdmin for adding OM admins, and  xSCOMManagementPack for installing OM Management Packs. |
| xSCSPF             | Updated          | Adds xSCSPFServerUpdate for updating and SPF server to Update Rollup 4. |
| xScSr              | Updated          | Adds the xSCSRServerUpdate resource for updating an SR server to Update Rollup 4 |
| xScVMM             | Updated          | Adds 3 new resources: xSCVMManagementServerUpdate, and xSCVMMConsoleUpdate for updating to Update Rollup 4, and xSCVMMAdmin for adding VMM admins |
| xSqlServer         | Updated          | Adds 5 new DSC resources for SQL failover clusters, and reporting services config. Also fixes an issue with SQL setup when using SQL mixed mode security. |
| xTimeZone          | New              | Set the Time Zone using DSC with this resource               |
| xWindowsRestore    | new              | Two new resources allow you to use DSC to configure system restore, create or remove a restore point |

The [xAzurePack][3] is a great addition and I will be using it right away! 🙂

You can download the new resource kit using PowerShell, whatelse!?

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-WebRequest -Uri 'https://gallery.technet.microsoft.com/DSC-Resource-Kit-All-c449312d/file/131371/1/DSC%20Resource%20Kit%20Wave%209%2012172014.zip' -OutFile "${env:TEMP}\DSC-Wave9.zip"
</pre>

[1]: http://blogs.msdn.com/b/powershell/archive/2014/12/17/another-holiday-present-from-the-powershell-team-dsc-reskit-wave-9.aspx
[2]: https://gallery.technet.microsoft.com/DSC-Resource-Kit-All-c449312d
[3]: https://gallery.technet.microsoft.com/xAzurePack-PowerShell-b25ed05f