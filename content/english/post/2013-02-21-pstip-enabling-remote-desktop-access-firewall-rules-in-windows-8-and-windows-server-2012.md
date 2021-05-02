---
title: '#PSTip Enabling remote desktop access firewall rules in Windows 8 and Windows Server 2012'
author: Ravikanth C
type: post
date: 2013-02-21T19:00:55+00:00
url: /2013/02/21/pstip-enabling-remote-desktop-access-firewall-rules-in-windows-8-and-windows-server-2012/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

In a couple of earlier posts, we looked at how we can [get firewall rules][1] and [add new rules][2]. In this post, we shall see an example of extending this knowledge to enable firewall rule for remote desktop access.

There is a built-in firewall rule that needs to enabled for allowing remote desktop access. Make a note that this is not about enabling remote desktop but ensuring that we allow remote desktop access in Windows Firewall.

First, let us see how we can check if the remote desktop firewall rule is enabled:

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallRule -DisplayName "Remote Desktop*" | Select DisplayName, Enabled
</pre>

When you run the above command, you will see two firewall rules &#8211; &#8220;Remote Desktop &#8211; User Mode (TCP-In)&#8221; and &#8220;Remote Desktop &#8211; User Mode (UDP-In)&#8221;.

We have to enable both these rules to ensure we allow remote desktop access through Windows Firewall.

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetFirewallRule -DisplayName "Remote Desktop*" | Set-NetFirewallRule -enabled true
</pre>

That is it! You will see that the remote desktop firewall rules are now enabled.

[1]: /2013/02/20/pstip-get-windows-firewall-rule-status-in-windows-8-and-server-2012/
[2]: /2013/02/19/pstip-adding-firewall-rules-in-windows-8-and-server-2012/