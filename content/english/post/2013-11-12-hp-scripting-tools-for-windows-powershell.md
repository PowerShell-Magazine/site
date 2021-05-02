---
title: HP Scripting Tools for Windows PowerShell
author: Shay Levy
type: post
date: 2013-11-12T14:31:07+00:00
url: /2013/11/12/hp-scripting-tools-for-windows-powershell/
categories:
  - News
  - HPE
  - Module Spotlight
tags:
  - HPE
  - Modules
  - News

---
HP has released a PowerShell module [HPiLOCmdlets,][1] designed to help IT experts to configure and manage HP iLO 3 and iLO 4 using Windows PowerShell. The HP iLO module is a set of a 110 functions that provides out-of-band automation from a Windows management workstation directly to iLO systems.

Features of the module include:

  * Sending commands to multiple iLOs
  * Finding iLO systems by searching IP addresses
  * Access to information available from iLO systems, including configuration settings, power state and settings, system health, IML and iLO event logs, and many more
  * Setting configurable iLO settings in scripts

**You cannot** update iLO firmware with this release.

The module requires [Windows Management Framework 3.0][2] and is supported on the following operating systems:

• Microsoft Windows 7 SP1

• Microsoft Windows 8

• Microsoft Windows Server 2008 R2 SP1

• Microsoft Windows Server 2012

An installation file is available for both 64-bit and 32-bit systems. A 21-page user guide can be downloaded from [HERE][3].

There&#8217;s one **major drawback** with the module&#8211;it cannot be discovered (on two x64 bit systems I tested on: Windows 7 and Windows 8.1). The installation creates the module under the _C:\Program Files\Hewlett-Packard\PowerShell\Modules\HPiLOCmdlets_ folder. The path of the folder is not visible during installation and the installation process doesn&#8217;t update the path in the _$env:PSModulePath_ variable.

Until this gets fixed you might want to save yourself some typing each time you want to use the module and add this line to your profile file so it can be easily discovered with _Get-Help, Get-Module, Get-Command_ or any other discovery cmdlet:

<pre class="brush: powershell; title: ; notranslate" title="">$env:PSModulePath+=";$env:ProgramFiles\Hewlett-Packard\PowerShell\Modules"
</pre>

[1]: http://h20566.www2.hp.com/portal/site/hpsc/public/psi/swdHome/?lang=en&cc=us&sp4ts.oid=5440658
[2]: (http://www.microsoft.com/en-us/download/details.aspx?id=34595
[3]: http://h20566.www2.hp.com/portal/site/hpsc/template.BINARYPORTLET/public/kb/docDisplay/resource.process/?spf_p.tpst=kbDocDisplay_ws_BI&spf_p.rid_kbDocDisplay=docDisplayResURL&javax.portlet.begCacheTok=com.vignette.cachetoken&spf_p.rst_kbDocDisplay=wsrp-resourceState%3DdocId%253Demr_na-c03958206-1%257CdocLocale%253Den_US&javax.portlet.endCacheTok=com.vignette.cachetoken