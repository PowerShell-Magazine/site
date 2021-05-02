---
title: '#PSTip How do I determine if my script is running in a RDP session?'
author: Shay Levy
type: post
date: 2013-01-22T19:00:54+00:00
url: /2013/01/22/pstip-how-do-i-determine-if-my-script-is-running-in-a-rdp-session/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
If you list the Environment drive, you&#8217;ll find a variable called _SESSIONNAME_. _SESSIONNAME_ is defined only if the Terminal Services system component is installed.

```
PS> Get-ChildItem env:\s*
Name                           Value
----                           -----
SystemDrive                    C:
SystemRoot                     C:\Windows
SESSIONNAME                    Console
```

The value of the variable is set to &#8216;Console&#8217; which means that you&#8217;re currently running directly on the local machine. However, when you initiate a remote desktop session to another machine, the value of the variable will be different, such as: &#8216;RDP-Tcp#0&#8217;.

Based on that, we can determine if our code is running inside a remote desktop session.

```
if($env:SESSIONNAME -eq 'Console')
{
    Write-Host "You are running on the local machine."
}
else
{
    Write-Host "You are in a Remote desktop session ('$env:SESSIONNAME')."
}
```

