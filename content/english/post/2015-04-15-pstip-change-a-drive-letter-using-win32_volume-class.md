---
title: 'PSTip: Change a drive letter using Win32_Volume class'
author: Jaap Brasser
type: post
date: 2015-04-15T16:00:29+00:00
url: /2015/04/15/pstip-change-a-drive-letter-using-win32_volume-class/
views:
  - 13433
post_views_count:
  - 2363
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Changing a drive letter of a removable drive or DVD drive is an easy task with Windows PowerShell. Using the Win32_Volume class and the Set-CimInstance cmdlet a drive letter can be reassigned. The following code will change the drive letter for the F: drive and change the drive letter to Z:

```powershell
$DvdDrive = Get-CimInstance -Class Win32_Volume -Filter "driveletter='F:'"
Set-CimInstance -InputObject $DvdDrive -Arguments @{DriveLetter="Z:"}
```


Alternatively this can also be executed in a single line of code by using the pipeline:

```powershell
Get-CimInstance -Query "SELECT * FROM Win32_Volume WHERE driveletter='F:'" | 
Set-CimInstance -Arguments @{DriveLetter="Z:"}
```


For more information about the Win32_Volume class have a look at the MSDN entry here:

[Win32_Volume Class][1]

[1]: https://msdn.microsoft.com/en-us/library/aa394515(v=vs.85).aspx