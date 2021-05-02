---
title: '#PSTip Select non-administrative shared folders using Win32_Share WMI class'
author: Jaap Brasser
type: post
date: 2014-07-24T18:00:04+00:00
url: /2014/07/24/pstip-select-non-administrative-shared-folders-using-win32_share-wmi-class/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI
---
**Note:** This tip required PowerShell 3.0 or above

The _Win32_Share_ WMI class can be used to enumerate all shares on the local or remote system. In this tip, _Get-CimInstance_ will be utilized to only return the non-administrative shared folders. This is possible because of the _Type_ property that is available on each shared folder. The Type property indicates whether a share is an administrative or a normal share. The table includes all eight available type numbers that the _Win32_Share_ class can return:

| **Type**                | **Description**   |
| ----------------------- | ----------------- |
| 0 (0x0)                 | Disk Drive        |
| 1 (0x1)                 | Print Queue       |
| 2 (0x2)                 | Device            |
| 3 (0x3)                 | IPC               |
| 2147483648 (0x80000000) | Disk Drive Admin  |
| 2147483649 (0x80000001) | Print Queue Admin |
| 2147483650 (0x80000002) | Device Admin      |
| 2147483651 (0x80000003) | IPC Admin         |

To get only non-administrative disk shares run the following command:

```
Get-CimInstance –Class Win32_Share –ComputerName server01 | Where-Object {$_.Type –eq 0}
```

Alternatively the _–Query_ parameter can also be used to query this class:

```
Get-CimInstance -Query "select * from Win32_Share where type=0" –ComputerName localhost
```

The _Get-CimInstance_ cmdlet is available in PowerShell 3.0 and later. The code posted in this tip can also be used in combination with the _Get-WmiObject_ cmdlet to get similar results while maintaining compatibility with PowerShell 1.0/2.0.

For more information in regards to the _Win32_Share_ class please see the following MSDN article:

<http://msdn.microsoft.com/en-us/library/aa394435(v=vs.85).aspx>