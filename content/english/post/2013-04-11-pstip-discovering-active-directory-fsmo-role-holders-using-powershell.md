---
title: '#PSTip Discovering Active Directory FSMO Role Holders using PowerShell'
author: Shay Levy
type: post
date: 2013-04-11T18:00:08+00:00
url: /2013/04/11/pstip-discovering-active-directory-fsmo-role-holders-using-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Active Directory defines five FSMO roles:

  * Schema master
  * Domain naming master
  * RID master
  * PDC master
  * Infrastructure master

The first two: schema master and the domain naming master are per-forest roles. There can be only one of each per forest. The other three: RID master, PDC master, and the infrastructure master are per-domain roles. Each domain has its own RID master, PDC master, and infrastructure master.

There are numerous ways and tools to help you get the information, including the [PowerShell AD module][1]. With the following code, you can get the forest and domain role holders of the  current user using PowerShell only.

<pre class="brush: powershell; title: ; notranslate" title="">[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain() | Select-Object *owner
[System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest() | Select-Object *owner
</pre>

[1]: http://technet.microsoft.com/en-us/library/ee617195.aspx