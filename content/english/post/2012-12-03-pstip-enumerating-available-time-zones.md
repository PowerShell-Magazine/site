---
title: '#PSTip Enumerating available time zones'
author: Ravikanth C
type: post
date: 2012-12-03T19:00:56+00:00
url: /2012/12/03/pstip-enumerating-available-time-zones/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Some applications or scripts might require the presence of a specific time zone on the system to be able to execute the actions defined by the application. For example, your script might have to adjust to the date/time format used in the script or consider daylight savings to ensure the script or application task scheduling is accurate. This is just one example of a time zone-aware application.

For the time zone-awareness, we need to be able to identify what time zones are available. We can do this using the [TimeZoneInfo][1] .NET class.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [TimeZoneInfo]::GetSystemTimeZones() | select Id, DisplayName
</pre>

[1]: http://msdn.microsoft.com/en-us/library/system.timezoneinfo.aspx