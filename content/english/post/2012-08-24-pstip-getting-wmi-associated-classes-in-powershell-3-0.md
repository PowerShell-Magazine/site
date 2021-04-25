---
title: '#PSTip Getting WMI associated classes in PowerShell 3.0'
author: Ravikanth C
type: post
date: 2012-08-24T18:00:42+00:00
url: /2012/08/24/pstip-getting-wmi-associated-classes-in-powershell-3-0/
views:
  - 7160
post_views_count:
  - 1452
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In Windows PowerShell 2.0, retrieving WMI associations and references required complex WMI queries. For example, if you wanted to get the IP address bound to a particular network adapter using WMI associations you would run:

<pre class="brush: powershell; title: ; notranslate" title="">$query = "ASSOCIATORS OF {Win32_NetworkAdapter.DeviceID=12} WHERE ResultClass=Win32_NetworkAdapterConfiguration"
Get-WmiObject -Query $query
</pre>

This certainly requires a good understanding of WMI Query Language and how to use different keywords in WQL.

Enter Windows PowerShell 3.0!

PowerShell 3.0 includes [CIM cmdlets][1] and the above example can be easily implemented using the new [Get-CimAssociatedInstance][2] cmdlet.

Let us see how:

<pre class="brush: powershell; title: ; notranslate" title="">$net = Get-CimInstance -ClassName Win32_NetworkAdapter -Filter "DeviceID=12"
Get-CimAssociatedInstance -InputObject $net -ResultClassName Win32_NetworkAdapterConfiguration
</pre>

That is it. Simple!

[1]: http://technet.microsoft.com/en-us/library/jj553783
[2]: http://technet.microsoft.com/en-us/library/jj590756