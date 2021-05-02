---
title: '#PSTip How to disable DNS registration with PowerShell'
author: Emin Atac
type: post
date: 2014-03-07T19:00:58+00:00
url: /2014/03/07/pstip-how-to-disable-dns-registration-with-powershell/
categories:
  - WMI
  - Tips and Tricks
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 2.0 or above.

Today, a colleague in the network team asked if we could disable the default DNS registration flag (shown below) because our computers are not allowed to update DNS servers directly.

![](/images/image0011.png)

As only Windows 7 computers are targeted, WMI is the correct way to go. We first need to find the WMI properties related to the DNS configuration of the network card.

```
(Get-WmiObject Win32_NetworkAdapter -Filter "NetEnabled=True").GetRelated('Win32_NetworkAdapterConfiguration') |
Format-List -Property *
```

![](/images/image0012.png)

The second step consists of using the _Get-Member_ cmdlet to discover the method designed to configure the DNS registration flag.

```
(Get-WmiObject Win32_NetworkAdapter -Filter "NetEnabled=True").GetRelated('Win32_NetworkAdapterConfiguration') |
Get-Member -MemberType Method
```


![](/images/image0041.jpg)

_&#8220;Invoking&#8221;_ the method without any parenthesis will then show the parameters to be passed to the method:

```
(Get-WmiObject Win32_NetworkAdapter -Filter "NetEnabled=True").
GetRelated('Win32_NetworkAdapterConfiguration').
SetDynamicDNSRegistration
```


![](/images/image006.jpg)

To test the method and its effect, the following can be done:

```
(Get-WmiObject Win32_NetworkAdapter -Filter "NetEnabled=True").GetRelated('Win32_NetworkAdapterConfiguration').SetDynamicDNSRegistration($false,$false)
```


Desktops usually have only one network card and the above code would work fine on PowerShell 2.0 in this case. But the above code will also produce an error if there’s more than one network card. The above code isn&#8217;t actually suitable for laptops. To get the code work on laptops with PowerShell 2.0, we need to pipe it to the _ForEach-Object_ cmdlet:

```
(Get-WmiObject -Class Win32_NetworkAdapter -Filter "NetEnabled=True").GetRelated('Win32_NetworkAdapterConfiguration') | ForEach-Object {
    $_.SetDynamicDNSRegistration($false,$false)
}
```


On PowerShell 3.0 the following would disable the DNS registration flag on all network cards because of the _implicit foreach_ feature

```
(Get-WmiObject Win32_NetworkAdapter -Filter "NetEnabled=True").GetRelated('Win32_NetworkAdapterConfiguration').SetDynamicDNSRegistration($false,$false)
```


And, if we had Windows 8/2012 or above, the correct way to go would have been to use the built-in cmdlets like this:

```
Get-NetIPConfiguration |
Get-NetConnectionProfile |
Where IPv4Connectivity -ne "NoTraffic" |
Set-DnsClient -RegisterThisConnectionsAddress:$false -Verbose
```


The above where filter can also be avoided like this:

```
Get-NetConnectionProfile -IPv4Connectivity Internet,Disconnected,LocalNetwork,Subnet -ErrorAction SilentlyContinue |
Set-DnsClient -RegisterThisConnectionsAddress:$false -Verbose
```


Last but not least, DNS records might not be automatically removed from DNS servers. You may need to follow additional steps from this knowledge based article whether your network card was configured to use a static or a dynamic IP address: <http://support.microsoft.com/kb/2933537/en-us>.