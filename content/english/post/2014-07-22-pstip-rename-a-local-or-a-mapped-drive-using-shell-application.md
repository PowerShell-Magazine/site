---
title: '#PSTip Rename a local or a mapped drive using Shell.Application'
author: Jaap Brasser
type: post
date: 2014-07-22T18:00:56+00:00
url: /2014/07/22/pstip-rename-a-local-or-a-mapped-drive-using-shell-application/
post_views_count:
  - 3213
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In the &#8216;[Create mapped network drive using WScript.Network][1]&#8216; post, it was shown how a mapped drive can be mounted. The mapped drive doesn’t initially have a friendly name&#8211;only the name of the shared folder and the server name will be shown. In order to rename the drive using PowerShell, the _Shell.Application_ COM object can be used. This code will allow you to rename a mapped network drive or any local disk:

<pre class="brush: powershell; title: ; notranslate" title="">$ShellObject = New-Object –ComObject Shell.Application
$DriveMapping = $ShellObject.NameSpace('Z:')
$DriveMapping.Self.Name = 'NewName'
</pre>

To make this shorter the following one-liner can be used to rename a local drive:

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object -ComObject Shell.Application).NameSpace('Z:').Self.Name='NewName'
</pre>

[1]: /2014/07/21/pstip-create-mapped-network-drive-using-wscript-network