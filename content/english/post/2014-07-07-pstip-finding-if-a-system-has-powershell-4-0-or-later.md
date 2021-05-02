---
title: '#PSTip Finding if a system has PowerShell 4.0 or later'
author: Ravikanth C
type: post
date: 2014-07-07T18:00:22+00:00
url: /2014/07/07/pstip-finding-if-a-system-has-powershell-4-0-or-later/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 3.0 or later.

I was looking for a way to find if systems in my environment have PowerShell 4.0 or not. I have a mix of systems with Windows Server 2012, Server 2012 R2, Server 2008 R2, Windows 7, Windows 8, and so on. If I were to check for the hotfix, I need to check different KB numbers for different operating systems. So, without knowing what OS the remote system is running, this is a little tricky and lengthy process.

Working through this, I found a simple method to detect if a remote system has PowerShell 4.0 or later. This method uses CIM cmdlets to find if the root\microsoft\windows\desiredstateconfiguration namespace exists on the remote system. DSC is a feature introduced in PowerShell 4.0. If this namespace exists on the remote system, we can safely conclude that the system has PowerShell 4.0 installed.

Here is the code in action:

```
Function Test-DscNamespace ($ComputerName = $env:COMPUTERNAME) {
    if (Get-CimInstance -ComputerName $ComputerName -Namespace root\Microsoft\Windows -ClassName __NAMESPACE -Filter "name='DesiredStateConfiguration'" -ErrorAction SilentlyContinue) {
         $true
    } else {
         $false
    }
}

$Servers = "Demo-Ad","WC7-1","WS8R2-1","WS8R2-2","WSR2-3-WMF50","WSR2-1"

foreach ($server in $Servers) {
    New-Object -TypeName PSObject -Property @{
        PSComputerName = $Server
        IsPowerShell4 = Test-DscNamespace -ComputerName $Server
    }
}
```

![](/images/ps4.png)

