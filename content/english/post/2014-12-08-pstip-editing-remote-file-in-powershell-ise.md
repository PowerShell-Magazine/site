---
title: '#PSTip Editing remote file in PowerShell ISE'
author: Ravikanth C
type: post
date: 2014-12-08T19:49:08+00:00
url: /2014/12/08/pstip-editing-remote-file-in-powershell-ise/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 5.0 or later.

If you have ever wondered how you can edit files on a remote system using Windows PowerShell, it is now possible with PowerShell 5.0 preview. This is, at the moment, possible only within PowerShell ISE.

<pre class="brush: powershell; title: ; notranslate" title="">Enter-PSSession -ComputerName computername
psEdit filename
</pre>

If you are wondering, what is psEdit, it existed in Windows PowerShell for a long time (since 2.0) and is used to add the specified file(s) to the current PowerShell tab in PowerShell ISE.

![](/images/psedit.png)

Here is a quick demo of remote file editing in PowerShell 5.0.

![](/images/PSEdit-WMF5.gif)

PowerShell console has _ise_ command that opens PowerShell ISE. However, it is not possible to perform remote file editing from PowerShell console prompt. The _ise_ command is mapped to _PowerShell_ise.exe_ and therefore it just creates a process on the remote session.