---
title: '#PSTip Resolve IP Address or a host name using .NET Framework'
author: Jaap Brasser
type: post
date: 2012-09-18T18:00:32+00:00
url: /2012/09/18/pstip-resolve-ip-address-or-a-host-name-using-net-framework/
views:
  - 18347
post_views_count:
  - 2831
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Using the GetHostEntry static method of the  .NET Framework System.Net.Dns class it  is possible to resolve IP address or a host name:

```
[System.Net.Dns]::GetHostEntry('computer1')
HostName                   Aliases   AddressList
--------                   -------   -----------
computer1.jaapbrasser.com  {}        {fe80::99e1:800d:6c35:e406%10, 10.0.0.1}
```

If the IP address is what is required  then the following PowerShell 3.0 syntax can be used:

<pre class="brush: powershell; title: ; notranslate" title="">[System.Net.Dns]::GetHostEntry('localhost').AddressList.IPAddressToString
</pre>

To filter only IPv4 addresses the following filter can be added:

<pre class="brush: powershell; title: ; notranslate" title="">[System.Net.Dns]::GetHostEntry('localhost').AddressList.IPAddressToString | Where-Object {$_ -match '\.'}
</pre>

To filter only IPv6 addresses just change the operator from -match to -notmatch. For a full list of the available methods in this .NET class the following command can be used:

<pre class="brush: powershell; title: ; notranslate" title="">[System.Net.Dns] | Get-Member -MemberType method -Static
</pre>

A full description of this .NET class can be found at MSDN:<http://msdn.microsoft.com/en-us/library/b8hth2dy>

### Editor's note

The formal way to filter IPv4 address only would be by comparing against the AddressFamily property:

<pre class="brush: powershell; title: ; notranslate" title="">[System.Net.Dns]::GetHostEntry('localhost').AddressList |
Where-Object {$_.AddressFamily -eq 'InterNetwork'} |
ForEach-Object {$_.IPAddressToString}
</pre>

For a complete list of AddressFamily values, see the [AddressFamily Enumeration page on MSDN][1]

[1]: http://msdn.microsoft.com/en-us/library/system.net.sockets.addressfamily.aspx