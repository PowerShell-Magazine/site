---
title: '#PSTip Copying folders using Copy-VMFile cmdlet in Windows Server 2012 R2 (Hyper-V)'
author: Ravikanth C
type: post
date: 2013-12-17T19:00:20+00:00
url: /2013/12/17/pstip-copying-folders-using-copy-vmfile-cmdlet-in-windows-server-2012-r2-hyper-v/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Tips and Tricks
  - Hyper-V

---
In an earlier article, I showed you [how to use the _Copy-VMFile_ cmdlet][1]. However, if you have paid attention and really used it, you will know that it cannot be used to copy a folder completely with its contents in a recursive manner.

Copying each and every single file isn&#8217;t a great experience, right?

Let us see how we can copy a complete folder to a VM.

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem C:\Scripts -Recurse -File | % { Copy-VMFile "WC7-1" -SourcePath $_.FullName -DestinationPath $_.FullName -CreateFullPath -FileSource Host }
</pre>

Observe the _-File_ switch parameter in the command above. It is needed because the _Copy-VMFile_ is capable of copying files only. So, if it encounters a folder, it simply fails. So, with the above command empty folders on the source path will not be created at the target because of the _-File_ switch parameter.

[1]: /2013/12/16/using-copy-vmfile-cmdlet-in-windows-server-2012-r2-hyper-v/