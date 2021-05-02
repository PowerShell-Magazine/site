---
title: '#PSTip How to start an elevated PowerShell from Windows Explorer'
author: Aleksandar Nikolic
type: post
date: 2013-06-25T18:00:02+00:00
url: /2013/06/25/pstip-how-to-start-an-elevated-powershell-from-windows-explorer/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

The next time you are working in a folder and need to open a Windows PowerShell console directly to that location, it would be nice to have &#8216;Open Windows PowerShell Here as Administrator&#8217; option on a context menu. To accomplish that, you&#8217;ll need to create a few Registry keys, and an elevated Windows PowerShell console will be just a right-click away from a folder icon, a drive icon, or inside of an open folder. To run this code, start Windows PowerShell with the &#8220;Run as administrator&#8221; option.

```
$menu = 'Open Windows PowerShell Here as Administrator'
$command = "$PSHOME\powershell.exe -NoExit -NoProfile -Command ""Set-Location '%V'"""

'directory', 'directory\background', 'drive' | ForEach-Object {
    New-Item -Path "Registry::HKEY_CLASSES_ROOT\$_\shell" -Name runas\command -Force |
    Set-ItemProperty -Name '(default)' -Value $command -PassThru |
    Set-ItemProperty -Path {$_.PSParentPath} -Name '(default)' -Value $menu -PassThru |
    Set-ItemProperty -Name HasLUAShield -Value ''
}
```

This is how the modified HKEY\_CLASSES\_ROOT\Directory\shell and HKEY\_CLASSES\_ROOT\Directory\Background\shell Registry keys look like (click for larger image):

![](/images/OpenPowerShellHereAsAdmin.png)

These Registry changes will create new entry on the context menu and this is how it looks when you right-click a folder icon, or inside of the open folder:

![](/images/ContextMenu.png)

_Context menu when you right-click a folder icon_

![](/images/ContextMenuBackground.png)

_Context menu when you right-click inside (on the background) of an open folder_