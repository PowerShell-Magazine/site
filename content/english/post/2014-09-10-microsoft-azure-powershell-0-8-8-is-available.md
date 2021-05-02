---
title: Microsoft Azure PowerShell 0.8.8 is available
author: Aleksandar Nikolic
type: post
date: 2014-09-10T21:25:16+00:00
url: /2014/09/10/microsoft-azure-powershell-0-8-8-is-available/
categories:
  - Azure
  - News
tags:
  - Azure
  - News

---
Just a few hours ago,  [Azure team released Microsoft Azure PowerShell 0.8.8][1]. The new release has a lot of improvements and added a support for today&#8217;s Azure update&#8211;Role-Based Access Control in Azure Preview Portal.

Here is the official change log for version 0.8.8:

  * Role-based access control support
  * Query role definition
  * Get-AzureRoleDefinition
  * Manage role assignment
  * New-AzureRoleAssignment
  * Get-AzureRoleAssignment
  * Remove-AzureRoleAssignment

  * Query Active Directory object
  * Get-AzureADUser
  * Get-AzureADGroup
  * Get-AzureADGroupMember
  * Get-AzureADServicePrincipal

  * Show user&#8217;s permissions on
  * Get-AzureResourceGroup
  * Get-AzureResource

  * Active Directory service principal login support in Azure Resource Manager mode
  * Add-AzureAccount -Credential -ServicePrincipal -Tenant

  * SQL Database auditing support in Azure Resource Manager mode
  * Use-AzureSqlServerAuditingSetting
  * Set-AzureSqlServerAuditingSetting
  * Set-AzureSqlDatabaseAuditingSetting
  * Get-AzureSqlServerAuditingSetting
  * Get-AzureSqlDatabaseAuditingSetting
  * Disable-AzureSqlServerAuditing
  * Disable-AzureSqlDatabaseAuditing

  * Other improvements
  * Virtual Machine DSC extension supports PSCredential as configuration argument
  * Virtual Machine Antimalware extension supports native JSON configuration
  * Storage supports creating storage account with different geo-redundant options
  * Traffic Manager supports nesting of profiles
  * Website supports configuring x32/x64 worker process
  * -Detail parameter on Get-AzureResourceGroup to improve performance
  * Major refactoring around account and subscription management

The error &#8220;_Add-AzureAccount: multiple\_matching\_tokens_detected: The cache contains multiple tokens satisfying the requirements. Call AcquireToken again providing more requirements (e.g. UserId)_&#8221; that some users have experienced with the Add-AzureAccount cmdlet, and were blocked to work with the Resource Manager in PowerShell, is fixed now.

The &#8220;Virtual Machine DSC extension supports PSCredential as configuration argument&#8221; improvement is covered in detail in <a href="http://blogs.msdn.com/b/powershell/archive/2014/09/10/secure-credentials-in-the-azure-powershell-desired-state-configuration-dsc-extension.aspx" title="Secure credentials in the Azure PowerShell Desired State Configuration (DSC) extension" target="_blank">today&#8217;s post on PowerShell team blog</a>.

[1]: https://github.com/Azure/azure-sdk-tools/releases