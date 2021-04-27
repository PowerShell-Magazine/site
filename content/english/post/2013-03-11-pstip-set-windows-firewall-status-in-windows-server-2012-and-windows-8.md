---
title: '#PSTip Set Windows Firewall status in Windows Server 2012 and Windows 8'
author: Ravikanth C
type: post
date: 2013-03-11T18:00:08+00:00
url: /2013/03/11/pstip-set-windows-firewall-status-in-windows-server-2012-and-windows-8/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

The _NetSecurity_ module in Windows Server 2012 and Windows 8 lets us configure Windows Firewall settings locally and remotely. The _Set-NetFirewallProfile _can be used to enable or disable the firewall.

First, let us see how we can enable firewall

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallProfile | Set-NetFirewallProfile -Enabled True
</pre>

To disable firewall, we can run

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallProfile | Set-NetFirewallProfile -Enabled False
</pre>

The above commands will change the firewall status on the local computer. How about remote systems? Simple, we use the _-CimSession_ parameter.

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallProfile -CimSession Server-01 | Set-NetFirewallProfile -Enabled False
</pre>

By default, the _Get-NetFirewallProfile_ cmdlet returns all Firewall profiles &#8212; Domain, Private, and Public. So, by sending the output of this to _Set-NetFirewallProfile_, we can either disable or enable all profiles at once. It is also possible to select a specified profile using the _-Name_ parameter.