---
title: '#PSTip Identifying the name of the PowerShell host'
author: Ravikanth C
type: post
date: 2012-11-16T19:00:57+00:00
url: /2012/11/16/pstip-identifying-the-name-of-the-powershell-host/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

$host automatic variable contains the details of the PowerShell host. For example, information such as the name of the host, UI culture information, etc. We can use this automatic variable to find the name of the PowerShell host.

<pre class="brush: powershell; title: ; notranslate" title="">$host.Name
</pre>

The output of the above snippet will be &#8220;ConsoleHost&#8221; for PowerShell.exe and &#8220;Windows PowerShell ISE Host&#8221; for PowerShell ISE.

When in a remote session (for example, using Enter-PSSession) or referring to $host.name inside the Invoke-Command&#8217;s script block, you will receive the host name as ServerRemoteHost.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt;Invoke-Command -ComputerName Server01 -ScriptBlock {$host.name}
ServerRemoteHost
</pre>

This is especially useful when your script needs to act in a different way in a remote session as compared to a local session.