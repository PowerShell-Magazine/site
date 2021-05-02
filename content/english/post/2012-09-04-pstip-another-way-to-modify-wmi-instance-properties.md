---
title: '#PSTip Another way to modify WMI instance properties'
author: Shay Levy
type: post
date: 2012-09-04T18:00:16+00:00
url: /2012/09/04/pstip-another-way-to-modify-wmi-instance-properties/
views:
  - 13633
  - 3991
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In the [previous tip][1] we showed you how to modify WMI object properties using Get-WmiObject cmdlet and the Put method. Today I want to show you another streamlined way to do the same using the Set-WmiInstance cmdlet.

The Set-WmiInstance cmdlet creates or updates an instance of an existing WMI class and writes the updates back to the WMI repository. So, if we wanted to modify the volume name of drive D:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Class Win32_LogicalDisk -Filter "DeviceID='D:'" |
Set-WmiInstance -Arguments @{VolumeName = "Data"}
</pre>

The Arguments parameter accepts a hashtable of name-value pair, property names, and the new values we want to set. To update multiple properties, delimit them with a semicolon @{Property1=&#8217;Value1&#8242;; Property=&#8217;Value2&#8242;}

**Note**: You must have administrative rights to update the properties and the console must be run elevated.

[1]: /2012/09/03/pstip-modifying-wmi-object-properties-using-get-wmiobject-cmdlet/ "#PSTip Modifying WMI Object properties using Get-WmiObject cmdlet"