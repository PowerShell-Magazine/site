---
title: '#PSTip Find PowerShell host process name'
author: Ravikanth C
type: post
date: 2012-11-15T19:00:50+00:00
url: /2012/11/15/pstip-find-powershell-host-process-name/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Every PowerShell host implements the $pid automatic variable which refers to the process ID of the PowerShell host process. Using it, we can find the name of the PowerShell host process within a script or a command.

<pre class="brush: powershell; title: ; notranslate" title="">(Get-Process -Id $pid).Name
</pre>

The above snippet outputs process name such as powershell and powershell\_ise for PowerShell.exe and PowerShell\_ISE.exe respectively. Or any other process name depending on the script editor (such as PowerShell Plus, etc) or the PowerShell host you have installed.