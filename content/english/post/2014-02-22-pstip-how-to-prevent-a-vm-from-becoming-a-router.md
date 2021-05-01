---
title: '#PSTip How to prevent a VM from becoming a Router'
author: Shay Levy
type: post
date: 2014-02-22T19:00:17+00:00
url: /2014/02/22/pstip-how-to-prevent-a-vm-from-becoming-a-router/
categories:
  - Hyper-V
  - Tips and Tricks
tags:  
  - Hyper-V
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Similarly to the Hyper-V security feature discussed [here][1], Hyper-V can also protect the environment from VMs pretending to act as Routers.

Router Guard allows you to specify whether the router advertisement and redirection messages from unauthorized VMs should be dropped. A malicious VM can send router advertisement messages or respond to another VM’s router solicitation messages to claim itself as the router.

Set the _RouterGuard_ parameter to_ &#8220;On&#8221; _to_ drop router messages sent from a VM. _Set it to_ &#8220;Off&#8221; _to allow the messages.

<pre class="brush: powershell; title: ; notranslate" title="">Set-VMNetworkAdapter -VMName VM1 -Name Public -RouterGuard On
</pre>

[1]: /2014/02/21/pstip-how-to-prevent-rouge-dhcp-servers-in-hyper-v/ "#PSTip How to prevent rogue DHCP servers in Hyper-V"