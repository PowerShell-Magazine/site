---
title: '#PSTip How to create a share in Windows Server 2012'
author: Aleksandar Nikolic
type: post
date: 2012-08-21T18:00:37+00:00
url: /2012/08/21/pstip-how-to-create-a-share-in-windows-server-2012/
views:
  - 15321
post_views_count:
  - 2135
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Quite easy, with a one-liner. Windows Server 2012 provides Windows PowerShell cmdlets and WMI objects to manage SMB file servers and SMB file shares. The SMB cmdlets are packaged into two modules called SmbShare and SmbWitness. These modules are automatically loaded (thanks to new module auto-loading feature) whenever you refer to any of the contained cmdlets. No upfront configuration is required. (Note: Check the output of the Get-Module command before and after you run the following one-liner to see that SmbShare module is loaded behind the scenes.)

<pre class="brush: powershell; title: ; notranslate" title="">New-SmbShare –Name MyShare –Path C:\Test –Description 'Test Shared Folder' –FullAccess Administrator –ReadAccess Everyone
</pre>

