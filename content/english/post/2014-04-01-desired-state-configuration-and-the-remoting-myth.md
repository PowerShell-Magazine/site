---
title: Desired State Configuration and the remoting myth
author: Ravikanth C
type: post
date: 2014-04-01T16:06:26+00:00
url: /2014/04/01/desired-state-configuration-and-the-remoting-myth/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
Windows PowerShell Desired State Configuration feature depends on PowerShell remoting. This is what we have been hearing and reading in the [WMF 4.0 release notes][1] too! Right?

It is a myth. Thanks to [Aleksandar][2] for shedding light on this. This is a myth like the other one that says we need PowerShell remoting for CIM cmdlets to work. CIM cmdlets and DSC just need the WinRM listeners and not PS remoting. The WinRM listeners are a requirement for DSC and PowerShell remoting creates them. This makes people think that PowerShell remoting is a requirement for DSC to work. Let us understand this in detail by looking at an example.

We will first go through the process of creating WinRM listeners and then make sure that we disable remoting. We, then, see how DSC configuration push just works without enabling PowerShell remoting on a target system. For this demo purpose, I chose Windows 8.1 x64 OS as the target OS. This system is joined to a domain and has Windows Remote Management disabled, by default.

### Verifying that Windows Remote Management is indeed disabled

This is straightforward. I just have to see if WinRM service on a target is system is running or not. For this, we can use the _Get-Service_ cmdlet.

![](/images/winrm1.png)

There are many ways to do this. You can use any of the following methods

```
#1. Using Set-WSManQuickConfig
Set-WSManQuickConfig

#2. Using winrm (do this at the console)
winrm quickconfig

#3. Enable PS Remoting
Enable-PSRemoting -Force
```

There are other methods such as using winrm command to create just the listener or using WSMAN provider to create the listener. On my target system, I just ran the _Set-WSManQuickConfig_ instead of the _Enable-PSRemoting_ as the release notes suggests.

![](/images/winrm2.png)

### Verify that WinRM listener is enabled

We can do this using the _Test-WSMan_ cmdlet. This tells us that the HTTP listener is created and functional.

![](/images/winrm3.png)

We can try and remote into the target system and make sure we really have remoting disabled.

![](/images/winrm61.png)

I have a sample DSC configuration that I want to push to the target system where remoting is not enabled. But, remember, I have the listeners still functioning on that.

```
Configuration WinRMDemo {
    Node WC81-1 {
       WindowsProcess ProcessDemo {
           Path="Notepad.exe"
           Arguments=""
           Ensure="Present"
       }
    }
}

WinRMDemo
Start-DscConfiguration -Path .\WinRMDemo -Wait -Verbose
```

![](/images/winrm7.png)

This is it. We pushed the DSC configuration without enabling remoting on the target system. Using the _Enable-PSRemoting_ cmdlet is probably the easiest way to create WinRM listeners and therefore people might be suggesting it as a method to create WinRM listeners. PowerShell remoting is more than just the WinRM listeners. It includes enabling session configurations and changing permissions to enable remote execution. PowerShell remoting is enabled by default on domain-joined Windows Server 2012 and Windows Server 2102 R2 systems, but it’s not a requirement for DSC.

[1]: http://www.microsoft.com/en-in/download/details.aspx?id=40855
[2]: https://twitter.com/alexandair/status/450695671180697600