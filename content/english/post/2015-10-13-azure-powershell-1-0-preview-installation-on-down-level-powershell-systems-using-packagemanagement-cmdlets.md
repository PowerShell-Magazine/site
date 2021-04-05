---
title: Azure PowerShell 1.0 Preview Installation on Down-level PowerShell Systems Using PackageManagement Cmdlets
author: Ravikanth C
type: post
date: 2015-10-13T14:43:07+00:00
url: /2015/10/13/azure-powershell-1-0-preview-installation-on-down-level-powershell-systems-using-packagemanagement-cmdlets/
views:
  - 8478
post_views_count:
  - 1585
categories:
  - Azure
  - Azure PowerShell
tags:
  - Azure
  - Azure PowerShell

---
If you have not seen this yet, [Azure PowerShell 1.0 is in preview][1]. If you have been using Azure Resource Manager cmdlets, there is a breaking change between 0.9.8 and 1.0. All Azure Resource Manager cmdlets in 1.0 have the AzureRM prefix. Also, there are different modules for different aspects of Azure management. One method you can get Azure PowerShell 1.0 Preview is to use the PackageManagement cmdlets. Azure blog quotes this saying:

![](/images/azposh1.png)

While the official blog post says that WMF 5.0 is required, it isn&#8217;t really necessary. This is because there is a [PackageManagement preview available for down-level PowerShell versions][2]. 🙂

Once you download and install this _PackageManagement_ preview on PowerShell 3.0 or 4.0 systems, you will get access to cmdlets such as _Find-Module_ and _Install-Module_.

Here is _$PSVersionTable_ from my system.

![](/images/azposh2.png)

```powershell
Find-Module -Name Azure*
```

![](/images/azposh3.png)

> **Note:** The above command will return Azure related modules written by community as well and not just Azure SDK team.

Let&#8217;s go ahead and install all modules with _AzureRM_ prefix (24 of them at the time of writing!)

```powershell
$AzureModules = Find-Module -Name AzureRM* | Select-Object -Expand Name
Install-Module -Name $AzureModules -Force
```


Or, you can install only _AzureRM_ module using the _Install-Module_ cmdlet and then use the _Install-AzureRM_ cmdlet to install remaining Azure Resource Manager modules. This cmdlet is an alias for the _Update-AzureRM_ cmdlet.

Finally, verify that all the _AzureRM_ modules got installed and the cmdlets are available.

![](/images/azposh4.png)

[1]: https://azure.microsoft.com/en-us/blog/azps-1-0-pre/
[2]: https://www.microsoft.com/en-us/download/details.aspx?id=49186