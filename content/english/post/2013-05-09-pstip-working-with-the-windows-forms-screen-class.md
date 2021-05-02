---
title: '#PSTip Working with the Windows.Forms.Screen Class'
author: Jaap Brasser
type: post
date: 2013-05-09T18:00:00+00:00
url: /2013/05/09/pstip-working-with-the-windows-forms-screen-class/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When working with Windows PowerShell it can be important to know the screen configuration and resolution of the current computer monitors. The _System.Windows.Forms.Screen_ class provides this information for every monitor attached to this computer. In order to use this class the _System.Windows.Forms.dll_ will be loaded first. After that we can view the _AllScreens_ property of this class:

```
PS> Add-Type -AssemblyName System.Windows.Forms
PS> [System.Windows.Forms.Screen]::AllScreens

BitsPerPixel : 32
Bounds       : {X=0,Y=0,Width=1920,Height=1200}
DeviceName   : \\.\DISPLAY1
Primary      : True
WorkingArea  : {X=0,Y=0,Width=1920,Height=1160}

BitsPerPixel : 32
Bounds       : {X=1920,Y=0,Width=1920,Height=1200}
DeviceName   : \\.\DISPLAY2
Primary      : False
WorkingArea  : {X=1920,Y=0,Width=1858,Height=1200}
```

This information can be especially handy when working with Windows Forms as it gives insight into how a desktop is configured and base the location or size of a form based on this information. Another use can be to determine where the taskbar is located.

```
[System.Windows.Forms.Screen]::AllScreens | ForEach-Object {
    if ($_.Bounds.Width -ne $_.WorkingArea.Width) {
        "Taskbar is placed in vertical position on $($_.DeviceName)"
    } elseif ($_.Bounds.Height -ne $_.WorkingArea.Height) {
        "Taskbar is placed in horizontal position on $($_.DeviceName)"
    } else {
        "Taskbar is not visible on $($_.DeviceName)"
    }
}

Taskbar is placed in horizontal position on \\.\DISPLAY1
Taskbar is placed in vertical position on \\.\DISPLAY2
```