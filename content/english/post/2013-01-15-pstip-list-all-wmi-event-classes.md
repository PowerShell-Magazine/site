---
title: '#PSTip List all WMI event classes'
author: Ravikanth C
type: post
date: 2013-01-15T19:11:30+00:00
url: /2013/01/15/pstip-list-all-wmi-event-classes/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 2.0 or above.

If you are familiar with Windows Management Instrumentation (WMI), there are different types of WMI queries possible. This includes data, event, and schema queries. In the context of today&#8217;s tip, we shall look at event queries. WMI events occur when a change happens in the WMI namespace being monitored. When an event that can be monitored by WMI occurs, an instance of the corresponding WMI event class is created, modified, or deleted. Starting PowerShell 2.0, we can use the _Register-WmiEvent_ cmdlet to subscribe to WMI events. We can use the _-Class _parameter to specify a WMI event class to subscribe to. For example, _Win32_ProcessStartTrace_ and _Win32_LocalTime_ are WMI event classes while _Win32_Process_ is not.

![](/images/eventclass-1024x162.png)

<p style="text-align: left;">
  As you see above, we see an error when the WMI class is not an event class. In fact, not all WMI classes are WMI event classes. So, how do we know which WMI classes are event classes?
</p>

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Query "SELECT * FROM meta_class WHERE __This ISA '__Event'"
</pre>

Simple! You can further filter this to show only Win32 WMI classes:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Query "SELECT * FROM meta_class WHERE (__This ISA '__Event') AND (__Class like 'win32%')"
</pre>