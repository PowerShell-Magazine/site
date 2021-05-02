---
title: '#PSTip Modifying WMI Object properties using Get-WmiObject cmdlet'
author: Ravikanth C
type: post
date: 2012-09-03T18:00:01+00:00
url: /2012/09/03/pstip-modifying-wmi-object-properties-using-get-wmiobject-cmdlet/
views:
  - 9172
post_views_count:
  - 2350
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Get-WmiObject and object modification! Sounds contradictory? Not really. That is the beauty of PowerShell and its object-based nature. Let us see an example to understand this.

<pre class="brush: powershell; title: ; notranslate" title="">$vol = Get-WmiObject -Class Win32_LogicalDisk -Filter "DeviceID='E:'"
$vol.VolumeName = "Memory Card"
</pre>

Now, setting the VolumeName to “Memory Card” isn’t enough.

<pre class="brush: powershell; title: ; notranslate" title="">$vol.Put()
</pre>

The Put() method ensures that the changes made to the object are written back. Make a note that you must have administrative rights to update some of the WMI object properties.