---
title: New version of the Windows Azure PowerShell
author: Aleksandar Nikolic
type: post
date: 2013-02-15T17:03:35+00:00
url: /2013/02/15/new-version-of-the-windows-azure-powershell/
categories:
  - News
tags:
  - News

---
Few days ago the new version (0.6.10) of the Windows Azure PowerShell has been released. The Windows Azure PowerShell provides IT Pros and developers with Windows PowerShell cmdlets for building, deploying, and managing Windows Azure services.

If you prefer Web Platform Installer, you can get the new version from <a href="http://www.windowsazure.com/en-us/downloads/" title="Windows Azure Downloads" target="_blank">Windows Azure Downloads</a> page. For those among you who like direct access to .msi file, there is a link on the <a href="https://github.com/WindowsAzure/azure-sdk-tools/downloads" title="Windows Azure SDK Tools" target="_blank">Windows Azure SDK Tools</a> GitHub page.

Here is the official change log for version 0.6.10 :

  * Upgrade to use PowerShell 3.0
  * Released source code for VM and Cloud Services cmdlets 
  * Added a few new cmdlets for cloud service scaffolding
  * Added Support for SAS in destination Uri for Add-AzureVhd 
  * Added -Confirm and -WhatIf support for Remove-Azure* cmdlets 
  * Added configurable startup task for Node.js and generic roles
  * Enabled emulator support when running roles with memcache 
  * Role-based cmdlets don&#8217;t require role name if run in a role folder
  * Added scenario test framework and started adding automated scenario tests
  * Multiple bug fixes