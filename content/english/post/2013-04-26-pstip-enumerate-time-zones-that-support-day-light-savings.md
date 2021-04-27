---
title: '#PSTip Enumerate Time Zones that support day light savings'
author: Ravikanth C
type: post
date: 2013-04-26T18:00:00+00:00
url: /2013/04/26/pstip-enumerate-time-zones-that-support-day-light-savings/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I was recently working on WPF based UI for one of my PowerShell modules and in the process, I had to figure out a way to find out if a selected timezone supports day light savings or not. I&#8217;d initially looked at [System.TimeZone][1] .NET class but could not find any relevant method or property.

The [GetSystemTimeZones()][2] method in System.TimeZoneInfo class was what I really needed. This method returns a set of time zones available on the local system and this includes a property called [SupportsDayLightSavingTime][3] property. We can use this property to filter out what we need!

<pre class="brush: powershell; title: ; notranslate" title="">[System.TimeZoneInfo]::GetSystemTimeZones() | Where { $_.SupportsDayLightSavingTime }
</pre>

[1]: http://msdn.microsoft.com/en-us/library/xe5cd9e6.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.timezoneinfo.getsystemtimezones.aspx
[3]: http://msdn.microsoft.com/en-us/library/system.timezoneinfo.supportsdaylightsavingtime.aspx