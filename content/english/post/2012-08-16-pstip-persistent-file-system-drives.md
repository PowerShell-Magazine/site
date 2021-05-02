---
title: '#PSTip Persistent file system drives'
author: Shay Levy
type: post
date: 2012-08-16T18:00:30+00:00
url: /2012/08/16/pstip-persistent-file-system-drives/
views:
  - 10964
post_views_count:
  - 1690
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In  [the previous tip][1] we showed you how  to get the next available drive letter using the Get-AvailableDriveLetter function when we want to create a new mapped network drive.

The Get-AvailableDriveLetter function is one of the commands in the Storage module in Windows 8 and Server 2012.

Starting with PowerShell 3.0, we can now map persistent drives using the enhanced New-PSDrive cmdlet. The following command uses the Get-AvailableDriveLetter function together with the New-PSDrive cmdlet and its -Persist parameter to map a persistent drive:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $letter = Get-AvailableDriveLetter -ReturnFirstLetterOnly
PS&gt; New-PSDrive -Name $letter -PSProvider FileSystem -Root \\Server01\Share -Persist
</pre>

To disconnect the drive:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Remove-PSDrive -Name $letter
</pre>

[1]: /2012/08/15/pstip-get-next-available-drive-letter/ "#PSTip Get next available drive letter"