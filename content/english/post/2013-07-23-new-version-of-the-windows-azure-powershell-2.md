---
title: New version of the Windows Azure PowerShell
author: Aleksandar Nikolic
type: post
date: 2013-07-23T13:22:43+00:00
url: /2013/07/23/new-version-of-the-windows-azure-powershell-2/
categories:
  - News
tags:
  - News

---
Few days ago the new version (0.6.17) of the Windows Azure PowerShell has been released. The Windows Azure PowerShell provides IT Pros and developers with Windows PowerShell cmdlets for building, deploying, and managing Windows Azure services.

If you prefer a Web Platform Installer, you can get the new version from <a title="Windows Azure Downloads" href="http://www.windowsazure.com/en-us/downloads/" target="_blank">Windows Azure Downloads</a> page. For those among you who like direct access to .msi file, there is a link on the <a title="Windows Azure SDK Tools" href="https://github.com/WindowsAzure/azure-sdk-tools/wiki/Downloads" target="_blank">Windows Azure SDK Tools</a> GitHub page or on the newly created <a title="Windows Azure PowerShell Releases" href="https://github.com/WindowsAzure/azure-sdk-tools/releases" target="_blank">Releases</a> page. This release is mostly focused on better PowerShell support for SQL Azure features.

Here is the official change log for version 0.6.17:

  * Upgraded Windows Azure SDK dependency from 1.8 to 2.0.
  * SQL Azure database CRUD cmdlets don&#8217;t require SQL auth anymore if the user owns the belonging subscription.
  * Get-AzureSqlDatabaseServerQuota cmdlet to get the quota information for a specified Windows Azure SQL Database Server.
  * SQL Azure service objective support 
      * Get-AzureSqlDatabaseServiceObjective cmdlet to a service objective for the specified Windows Azure SQL Database Server.
      * Added -ServiceObjective parameter to Set-AzureSqlDatabase to set the service objective of the specified Windows Azure SQL database.
  * Fixed a Get-AzureWebsite local caching issue. Now Get-AzureWebsite will always return the up-to-date web site information.