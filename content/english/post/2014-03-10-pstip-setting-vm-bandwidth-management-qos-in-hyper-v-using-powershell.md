---
title: '#PSTip Setting VM Bandwidth Management QoS in Hyper-V using PowerShell'
author: Shay Levy
type: post
date: 2014-03-10T18:00:10+00:00
url: /2014/03/10/pstip-setting-vm-bandwidth-management-qos-in-hyper-v-using-powershell/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - PowerShell

---
**Note**: This tip requires PowerShell 3.0 or above.

<span style="line-height: 1.5em;">The bandwidth utilized by a VM (QoS = Quality of Service ) can be controlled via special parameters of the </span>virtual network card for a given VM. This settings control the amount of bandwidth (minimum/maximum) a VM can use on the virtual switch and is available on a per network adapter basis in each virtual machine.

Bandwidth minimum guarantees the amount of bandwidth reserved and Bandwidth maximum caps the amount of bandwidth a VM can consume (in bits per second). To disable this feature, set it to 0 (zero).

This example sets the maximum bandwidth for VM1 to 200 Mbps

<pre class="brush: powershell; title: ; notranslate" title="">Set-VMNetworkAdapter -VMName VM1 -MaximumBandwidth 200000000
</pre>