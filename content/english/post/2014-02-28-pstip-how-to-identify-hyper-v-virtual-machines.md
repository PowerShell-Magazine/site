---
title: '#PSTip How to identify Hyper-V virtual machines'
author: Shay Levy
type: post
date: 2014-02-28T19:00:02+00:00
url: /2014/02/28/pstip-how-to-identify-hyper-v-virtual-machines/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

<span style="line-height: 1.5em;">There are several ways to test if a computer object is a virtual machine. </span>Most, if not all, methods require a connection to each computer, usually by checking a WMI property. If the computer you are targeting is not available, this information will not be available as well.What if you wanted to get the information from a single location, such as Active Directory? When it comes to Hyper-V based VMs, you can.

When a Hyper-V virtual machine is added to Active Directory, a _serviceConnectionPoint_ (SCP) object called _&#8220;Windows Virtual Machine&#8221; _is created under the computer object account (the object is created by the Hyper-V Integration service _&#8220;Hyper-V Heartbeat Service&#8221;_). We can use the _&#8220;Windows Virtual Machine&#8221;_ object to identify Windows Hyper-V virtual machines.

```
PS> Get-ADObject -Filter {objectClass -eq 'serviceConnectionPoint' -and Name -eq 'Windows Virtual Machine'}
DistinguishedName                         Name                      ObjectClass             ObjectGUID

-----------------                         ----                      -----------             ----------
CN=Windows Virtual Machine,CN=server1... Windows Virtual Machine   serviceConnectionPoint  d4d935cf-1310-4b6b-967d-469a9da1a88d
CN=Windows Virtual Machine,CN=server2... Windows Virtual Machine   serviceConnectionPoint  e60feb40-794a-4237-99af-1b1ef33f2984
CN=Windows Virtual Machine,CN=server3... Windows Virtual Machine   serviceConnectionPoint  53107864-ae4f-478c-a76c-fc0675a6d1b8
(...)
```

Getting the virtual machine name is a bit tricky. You could parse the machine name from the _DistinguishedName_ property but there&#8217;s an easier way. If you include the _CanonicalName_ property you can quickly split it and get the name

```
Get-ADObject -Filter {objectClass -eq 'serviceConnectionPoint' -and Name -eq 'Windows Virtual Machine'} -Properties CanonicalName | ForEach-Object{
    $_.CanonicalName.Split('/')[-2]
}

Server1
Server2
Server3
(...)
```