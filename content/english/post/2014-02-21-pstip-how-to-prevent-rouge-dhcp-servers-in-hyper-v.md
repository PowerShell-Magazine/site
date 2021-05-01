---
title: '#PSTip How to prevent rogue DHCP servers in Hyper-V'
author: Shay Levy
type: post
date: 2014-02-21T19:00:41+00:00
url: /2014/02/21/pstip-how-to-prevent-rouge-dhcp-servers-in-hyper-v/
categories:
  - Tips and Tricks
  - Hyper-V
tags:
  - Tips and Tricks
  - Hyper-V

---
**Note**: This tip requires PowerShell 3.0 or above.

In a DHCP environment, it is possible for a rogue DHCP server to respond to client DHCP requests and provide incorrect address and configuration information. A rogue DHCP server could be used to redirect traffic for malicious purposes.

Hyper-V Virtual Switches in Windows Server 2012 has a new security feature called DHCP Guard. It drops DHCP server messages from unauthorized VMs pretending to be DHCP server. DHCP Guard allows you to specify whether DHCP server messages coming from a VM should be dropped.

The following command prevents a VM from becoming a rogue DHCP server by turning DHCPGuard _&#8220;On&#8221;_. To turn it off, set DHCPGuard to _&#8220;Off&#8221;_.

<pre class="brush: powershell; title: ; notranslate" title="">Set-VMNetworkAdapter -VMName VM1 -DhcpGuard On
</pre>