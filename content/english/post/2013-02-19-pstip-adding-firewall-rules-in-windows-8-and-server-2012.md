---
title: '#PSTip Adding Firewall rules in Windows 8 and Server 2012'
author: Ravikanth C
type: post
date: 2013-02-19T19:00:30+00:00
url: /2013/02/19/pstip-adding-firewall-rules-in-windows-8-and-server-2012/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Before the release of Windows Server 2012 and Windows 8, adding rules to Windows Firewall required a painful approach of using the firewall COM object.

In Windows Server 2012 and Windows 8 operating systems, there is a new cmdlet called [_New-NetFirewallRule_][1]. This cmdlet provides a way to add new firewall rules.

The following example shows how to use this cmdlet to enable inbound traffic to port 80 on the local system.

<pre class="brush: powershell; title: ; notranslate" title="">New-NetFirewallRule -DisplayName "Allow Port 80" -Direction Inbound -LocalPort 80 -Protocol TCP
-Action Allow
</pre>

[1]: http://technet.microsoft.com/en-us/library/jj554908.aspx