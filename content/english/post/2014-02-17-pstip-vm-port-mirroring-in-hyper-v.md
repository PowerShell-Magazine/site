---
title: '#PSTip VM Port mirroring in Hyper-V'
author: Shay Levy
type: post
date: 2014-02-17T19:00:20+00:00
url: /2014/02/17/pstip-vm-port-mirroring-in-hyper-v/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Tips and Tricks
  - Hyper-V

---
**Note**: This tip requires PowerShell 3.0 or above.

In a physical switch environment, all traffic from selected ports can be duplicated and copied to a mirror port for capture and analysis, such as network diagnostic of a VM boot process or any network-related issues.

Starting in Windows Server 2012, port mirroring can be enabled on virtual switches as well. We can designate a vSwitch port as a monitoring port, and direct the vSwitch traffic going through this port to a specific VM.

The configuration is twofold&#8211;changes are needed on both, the source and destination VMs. This can be set under the Advanced Features page of the VMs&#8217; network card.

![](/images/portmirror.png)

The following commands sets port mirroring on two VMs using the _Set-VMNetworkAdapter_. VM1 acts as the source VM, every packet sent or received by VM1 (on all all virtual network adapters) will be mirrored to VM2 virtual network card named _Public_ . When you open a network monitor application on VM2, you will see captured traffic from both VMs.

<pre class="brush: powershell; title: ; notranslate" title="">Set-VMNetworkAdapter -VMName VM1 -PortMirroring Source
Set-VMNetworkAdapter -VMName VM2 -Name Public -PortMirroring Destination
</pre>

Note that you can set multiple VMs as source machines and all their traffic will be copied to the destination VM.