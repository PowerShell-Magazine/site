---
title: '#PSTip Controlling traffic of a VM network adapter in Hyper-V'
author: Shay Levy
type: post
date: 2014-02-24T19:00:12+00:00
url: /2014/02/24/pstip-controlling-traffic-of-a-vm-network-adapter-in-hyper-v/
categories:  
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Using the _Add-VMNetworkAdapterAcl_ cmdlet we can create ACLs (firewall-like rule) that applies to the traffic through a virtual machine network adapter. We can use it to allow or block traffic to or from specific sources by using IP addresses (including a range of addresses) or MAC addresses.

ACL rules apply to Hyper-V switch ports and currently can only be set using PowerShell. They control whether a packet is allowed or denied on the way into or out of the VM. Multiple port ACLs can be configure for a Hyper-V switch port.

The following command will deny inbound and outbound traffic from VM1 to the remote IP address 192.168.0.1

```
Add-VMNetworkAdapterAcl –VMName VM1 -RemoteIPAddress 192.168.0.1 -Direction
Both -Action Deny
```


To remove the rule:

```
Remove-VMNetworkAdapterAcl -VMName VM1 -RemoteIPAddress 192.168.0.1 -Action Deny -Direction Both
```

