---
title: '#PSTip Configuring trunk mode in Hyper-V'
author: Jan Egil Ring
type: post
date: 2014-08-06T18:00:29+00:00
url: /2014/08/06/configuring-trunk-mode-in-hyper-v/
categories:
  - Tips and Tricks
  - Hyper-V
tags:
  - Tips and Tricks
  - Hyper-V

---
With the introduction of Hyper-V in Windows Server 2012, the [Hyper-V Extensible Switch][1] became available providing more functionality than previous versions. One of the new features was support for VLAN trunk mode, making it possible for a virtual machine to see traffic from multiple VLANs. This might be a need for virtual machines running network services such as a reverse proxy.

Configuring trunk mode for a virtual network adapter is not supported from the Hyper-V Manager; only a single VLAN can be configured:

![](/images/Hyper-V_Manager_VLAN1.png)

In order to configure trunk mode, we can use the Hyper-V PowerShell module:

```
Set-VMNetworkAdapterVlan –VMName DemoVM –Trunk –AllowedVlanIdList 1000-1050 –NativeVlanId 1000
```


The above example will enable the virtual machine DemoVM to send and receive traffic on VLAN 1000 to 1050. If no VLAN is specified in the network packet, it will be processed on VLAN 1000.

[1]: http://technet.microsoft.com/en-us/library/hh831823.aspx