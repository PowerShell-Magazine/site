---
title: '#PSTip Detecting if a certain process is elevated'
author: Stefan Stranger
type: post
date: 2013-03-29T18:00:17+00:00
url: /2013/03/29/pstip-detecting-if-a-certain-process-is-elevated/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0

There was a question recently in the PowerShell MVP mailing list about how to detect if a certain process is elevated. There was a suggestion to use an external tool like the [Sysinternals tool AccessChk][1], but I&#8217;ve tried to find a way to use PowerShell.

My initial though was to look if WMI would show some properties which I could use to detect if a process is elevated. In PowerShell 3.0 I now use the _Get-CimInstance_ cmdlet to retrieve WMI information.

If you start, for example, an elevated Command Prompt (cmd.exe), _Win32_Process_ class properties retrieved in a non-elevated PowerShell session would not be the same as the properties retrieved in an elevated PowerShell session.

_Win32_Process_ class properties in a non-elevated PowerShell session for elevated Command Prompt.

![](/images/image1tip.png)

_Win32_Process_ class properties in an elevated PowerShell session for elevated Command Prompt.

![](/images/image3tip.jpg)

If you can see the CommandLine or ExecutablePath properties in a non-elevated PowerShell session for a process you know, this process is not started elevated.

[Kirk Munro][2] improved this solution by using the _Get-Process_ cmdlet instead of the _Get-CimInstance_ cmdlet. He looks for the Path and Handle properties of the process to detect if a process is elevated or not.

![](/images/image4tip.jpg)

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process |
Add-Member -Name Elevated -MemberType ScriptProperty -Value {if ($this.Name -in @('Idle','System')) {$null} else {-not $this.Path -and -not $this.Handle} } -PassThru |
Format-Table Name,Elevated
</pre>

Here we filter on all processes except the “Idle” and “System” processes and we check if we see the Path and Handle properties and finally use the Add-Member cmdlet to add a custom property (Name) to an instance of the Windows PowerShell object.

**<a href="http://104.131.21.239/wp-content/uploads/2013/03/image004.jpg" rel="lightbox[6130]"><img alt="image004" src="http://104.131.21.239/wp-content/uploads/2013/03/image004.jpg" width="624" height="148" /></a>**

[1]: http://technet.microsoft.com/en-us/sysinternals/bb664922.aspx
[2]: https://twitter.com/poshoholic