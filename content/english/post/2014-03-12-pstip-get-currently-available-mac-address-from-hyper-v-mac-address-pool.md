---
title: '#PSTip Get currently available MAC address from Hyper-V MAC address pool'
author: Ravikanth C
type: post
date: 2014-03-12T18:00:15+00:00
url: /2014/03/12/pstip-get-currently-available-mac-address-from-hyper-v-mac-address-pool/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
I have written scripts to automate VM deployments across a farm of Hyper-V servers and I prefer using static MAC addresses for all my workload virtual machines. So, it is important for me to find the available MAC address so that I can start incrementing from there.

The [Msvm_VirtualSystemManagementServiceSettingData][1] WMI class in the root\Virtualization\v2 namespace gives the maximum and minimum MAC addresses within the pool.

![](/images/mac.png)

However, the currently available MAC address is not given by this class. For this, we need to look into the Registry on the Hyper-V host at HKLM\Software\Microsoft\Windows NT\CurrentVersion\Virtualization\Worker. The _CurrentMacAddress_ value provides the MAC address that can be assigned to a VM.

```
$Path = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Worker'
$CurrentAddress = Get-ItemProperty -Path $Path -Name CurrentMacAddress
[System.BitConverter]::ToString($CurrentAddress.CurrentMacAddress)
```

![](/images/mac2.png)


[1]: http://msdn.microsoft.com/en-us/library/cc136941(v=vs.85).aspx