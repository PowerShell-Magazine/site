---
title: '#PSTip Resolving IP addresses with WMI'
author: Shay Levy
type: post
date: 2013-01-07T19:00:35+00:00
url: /2013/01/07/pstip-resolving-ip-addresses-with-wmi/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI
---
When we need to resolve addresses, we usually use the System.Net.Dns .NET class methods.  In addition to the .NET method, we can also use WMI class&#8211;[Win32_PingStatus][1]&#8211;to achieve this. The _Test-Connection_ cmdlet is great for checking if a host is up. You can pass a name or an IP address and send ICMP echo request packets (&#8220;pings&#8221;) to one or more computers. When you ping using a name, the result also includes its IP address.

```
PS> Test-Connection -ComputerName LOKI -Count 1
Source        Destination     IPV4Address      IPV6Address   Bytes    Time(ms)
------        -----------     -----------      -----------   -----    --------
SHAYPC        LOKI            10.10.10.10                    32       0
```

However, when you use an IP, you don&#8217;t get back the name. _Test-Connection_ relies on the _Win32_PingStatus_ class. There is an option available on the class properties to resolve the name. Once we set the _ResolveAddressNames_ property to _True,_ an attempt to resolve the address is made_._ If it succeeds, _ _the _ProtocolAddressResolved_ property is populated with the name of the  target.

```
PS> Get-WmiObject Win32_PingStatus -Filter "Address='10.10.10.10' AND ResolveAddressNames='true'" | Select-Object IPV4Address,ProtocolAddressResolved

IPV4Address   ProtocolAddressResolved
-----------   -----------------------
10.10.10.10   LOKI
```

It would be great if the _Test-Connection_ cmdlet had a _ResolveAddressNames _switch parameter instead of having to craft a WMI request. Like the idea? Would you like to see this in future releases of PowerShell? Add your vote to [this suggestion][2].

[1]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa394350(v=vs.85).aspx
[2]: https://connect.microsoft.com/PowerShell/feedback/details/759607/test-connection-resolving-address-names