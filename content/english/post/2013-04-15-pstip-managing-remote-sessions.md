---
title: '#PSTip Managing Remote Sessions'
author: Chad Miller
type: post
date: 2013-04-15T18:00:37+00:00
url: /2013/04/15/pstip-managing-remote-sessions/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires Windows PowerShell 2.0 or above.

The _Get-WSManInstance_ cmdlet can be used to view the sessions that are connected to a remote computer. You can run this cmdlet from any client that&#8217;s running PowerShell 2.0 or higher. Getting the number of WinRM sessions by user provides interesting information, especially when you are troubleshooting fan-in scenarios.

```
PS> Get-WSManInstance -ConnectionURI http://myserver.mydomain.com:5985/wsman shell -Enumerate

rsp             : http://schemas.microsoft.com/wbem/wsman/1/windows/shell
lang            : en-US
ShellId         : 75A89E0E-E6E5-477D-AD0F-DAD6706CC236
ResourceUri     : http://schemas.microsoft.com/powershell
Owner           : Mydomain\user1
ClientIP        : 192.168.2.11
ProcessId       : 5800
IdleTimeOut     : PT180.000S
InputStreams    : stdin pr
OutputStreams   : stdout
BufferMode      : Block
State           : Connected
ShellRunTime    : P0DT0H17M7S
ShellInactivity : P0DT0H0M7S
MemoryUsed      : 134MB
ChildProcesses  : 0

rsp             : http://schemas.microsoft.com/wbem/wsman/1/windows/shell
lang            : en-US
ShellId         : C334FE90-8CA7-4F26-8516-084EF68A2F32
ResourceUri     : http://schemas.microsoft.com/powershell
Owner           : 92.168.2.11
ClientIP        : ***
ProcessId       : 9592
IdleTimeOut     : PT180.000S
InputStreams    : stdin pr
OutputStreams   : stdout
BufferMode      : Block
State           : Connected
ShellRunTime    : P0DT0H22M25S
ShellInactivity : P0DT0H0M24S
MemoryUsed      : 91MB
ChildProcesses  : 0
(...)
```

Not only can you see the remote connections, but you can also &#8220;kill&#8221; connections by using _Remove-WSManInstance_. In this example, I&#8217;m using the command to kill the second session:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Remove-WSManInstance -ConnectionURI http://myserver.mydomain.com:5985/wsman shell @{ShellID="C334FE90-8CA7-4F26-8516-084EF68A2F32"}
</pre>

If you want to remove all of the sessions, you can restart the WinRM service, as shown in the next example:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Restart-Service -Name WinRM
</pre>

You can find more information about managing remote sessions [HERE][1].

[1]: http://blogs.msdn.com/b/powershell/archive/2009/06/04/managing-remote-sessions.aspx