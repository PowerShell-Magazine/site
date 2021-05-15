---
title: '#PSTip Get all WMI classes with methods'
author: Ravikanth C
type: post
date: 2012-11-08T19:00:35+00:00
url: /2012/11/08/pstip-get-all-wmi-classes-with-methods/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 2.0 or above.

When working with WMI and PowerShell, I often find it necessary to quickly generate a list of methods available in a WMI class. One way to do this is to examine the WMI class meta data. Let us see how:

```powershell
Get-WmiObject -Query 'Select * From Meta_Class WHERE __Class LIKE "win32%"' |
Where-Object { $_.PSBase.Methods } |
Select-Object Name, Methods
```


This will list all Win32 WMI classes with methods.

In Windows PowerShell 3.0, the same can be done using:

```powershell
Get-CimClass -ClassName win32* | where {$_.CimClassMethods} | select CimClassName,CimClassMethods
```


or

```powershell
Get-CimClass -ClassName win32* | where CimClassMethods -ne $null  | select CimClassName,CimClassMethods
```

