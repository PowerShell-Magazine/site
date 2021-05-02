---
title: '#PSTip Get system power information'
author: Shay Levy
type: post
date: 2012-10-18T18:00:03+00:00
url: /2012/10/18/pstip-get-system-power-information/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
With the PowerStatus class we can quickly get the information status of the current system. The PowerStatus class represents information about the current AC line power status, battery charging status, and battery charge status. To use PowerStatus class we first need to load the System.Windows.Forms assembly  (in ISE it is already loaded so you skip the Add-type command):

```
PS> Add-Type -Assembly System.Windows.Forms
PS> [System.Windows.Forms.SystemInformation]::PowerStatus

PowerLineStatus      : Offline
BatteryChargeStatus  : High
BatteryFullLifetime  : -1
BatteryLifePercent   : 0.76
BatteryLifeRemaining : 9500
```

Here&#8217;s a breakdown of the properties of the class:

### PowerLineStatus

Gets the current system power status. Indicates whether the system power is online, or that the system power status is unknown. Possible values:

  * Offline &#8211; The system is offline.
  * Online &#8211; The system is online.
  * Unknown &#8211; The power status of the system is unknown.

### BatteryChargeStatus

Gets the current battery charge status. Possible values:

  * High -Indicates a high level of battery charge.
  * Low &#8211; Indicates a low level of battery charge.
  * Critical &#8211; Indicates a critically low level of battery charge.
  * Charging &#8211; Indicates a battery is charging.
  * NoSystemBattery &#8211; Indicates that no battery is present.
  * Unknown &#8211; Indicates an unknown battery condition.

### BatteryFullLifetime

Gets the reported full charge lifetime of the primary battery power source in seconds. The reported number of seconds of battery life available when the battery is fully charged, or -1 if the battery life is unknown.

### BatteryLifePercent

Gets the approximate amount of full battery charge remaining. The approximate amount, from 0.0 to 1.0, of full battery charge remaining.

### BatteryLifeRemainingGets

The approximate number of seconds of battery life remaining, or –1 if the approximate remaining battery life is unknown. To get the value in hours:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; 9500/60/60
2.63888888888889
</pre>