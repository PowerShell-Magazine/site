---
title: '#PSTip Resolving an IP Address to host name and vice versa using .NET'
author: Ravikanth C
type: post
date: 2013-01-08T19:00:23+00:00
url: /2013/01/08/pstip-resolving-an-ip-address-to-host-name-and-vice-versa-using-net/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 1.0 or above.

In an earlier tip, we showed you how to [resolve IP addresses using WMI][1]. In PowerShell, there is always more than one way to achieve anything. So, in today&#8217;s tip, let us see how to use System.Net.Dns .NET class to achieve the same. In this .NET class, the [GetHostEntry()][2] method can be used to resolve IP address to host name and vice versa.

Here is how we use this method.

```
PS> [Net.DNS]::GetHostEntry("server01")
HostName          Aliases          AddressList
--------          -------          -----------
server01          {}               {192.94.21.28}

PS C:\> [Net.DNS]::GetHostEntry("192.94.21.28")
HostName         Aliases          AddressList
--------         -------          -----------
server01         {}               {}
```

[1]: /2013/01/07/pstip-resolving-ip-addresses-with-wmi/
[2]: http://msdn.microsoft.com/en-US/library/system.net.dns.gethostentry(v=vs.80).aspx