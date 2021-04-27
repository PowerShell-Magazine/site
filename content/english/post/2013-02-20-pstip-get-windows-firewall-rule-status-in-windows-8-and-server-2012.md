---
title: '#PSTip Get Windows Firewall rule status in Windows 8 and Server 2012'
author: Ravikanth C
type: post
date: 2013-02-20T19:00:01+00:00
url: /2013/02/20/pstip-get-windows-firewall-rule-status-in-windows-8-and-server-2012/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

In an earlier tip, we looked at how we can [add firewall rules in Windows 8 or Windows Server 2012][1]. In today&#8217;s tip, we shall look  at how we can get the firewall rules from local and remote systems in Windows 8 and Windows Server 2012 systems.

We can use the [_Get-NetFirewallRule_][2] cmdlet to achieve this. First, let us see how we can use this cmdlet on the local system.

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallRule -All
</pre>

The above command will list all available Firewall rules irrespective of their state (enabled or disabled) or action (allowed or denied). To filter this further to only enabled firewall rules, we can run:

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallRule -Enabled True
</pre>

We can filter this further and retrieve only the rules that are enabled and are set to allow.

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallRule -Enabled True -Action Allow
</pre>

So, how do we use this to retrieve the rules from a remote system? Simple, we need to use a computer name string or a CIM session object as an argument to the _-CimSession_ parameter of _Get-NetFirewallRule_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">$cimSession = New-CimSession -ComputerName Server-03
Get-NetFirewallRule -CimSession $cimSession -Enabled True -Action Allow
</pre>

Or

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallRule -CimSession Server-03 -Enabled True -Action Allow
</pre>

[1]: /2013/02/19/pstip-adding-firewall-rules-in-windows-8-and-server-2012/
[2]: http://technet.microsoft.com/en-us/library/jj554860.aspx