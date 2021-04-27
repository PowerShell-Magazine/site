---
title: '#PSTip Working with ISO files'
author: Shay Levy
type: post
date: 2013-01-02T19:00:18+00:00
url: /2013/01/02/pstip-working-with-iso-files/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 and Windows 8 or above.

Starting with Windows 8, mounting virtual hard disks or ISO files is not an issue anymore. The operation is built into the system. When mounted, a virtual hard disk or ISO file gets a drive letter assigned to it and it appears as a normal disk drive in Windows Explorer. To mount ISO files, simply double click them. To dismount, right click the drive and chose &#8216;Eject&#8217;.

In PowerShell, we can manage those drives via the Storage module and its _*-DiskImage_ cmdlets. For example, to mount an ISO image:

```
PS> Mount-DiskImage -ImagePath 'D:\ISO\en_windows_8_ent_x64.iso' -PassThru

Attached          : False
BlockSize         : 0
DevicePath        :
FileSize          : 3490912256
ImagePath         : D:\ISO\en_windows_8_ent_x64.iso
LogicalSectorSize : 2048
Number            :
Size              : 3490912256
StorageType       : 1
PSComputerName    :
```

By default, _Mount-DiskImage_ does not generate any output, so add the _PassThru_ switch to get a result.

```
PS> Dismount-DiskImage -ImagePath 'D:\ISO\en_windows_8_ent_x64.iso' -PassThru

Attached          : True
BlockSize         : 0
DevicePath        : \\.\CDROM1
FileSize          : 3490912256
ImagePath         : D:\ISO\en_windows_8_ent_x64.iso
LogicalSectorSize : 2048
Number            : 1
Size              : 3490912256
StorageType       : 1
PSComputerName    :
```

How about getting a list of mounted files? Let&#8217;s try the _Get-DiskImage_ cmdlet.

```
PS> Get-DiskImage

cmdlet Get-DiskImage at command pipeline position 1
Supply values for the following parameters:
ImagePath[0]:
```

Bummer! Instead of getting mounted images we are prompted to enter the path of the image file! So, to bypass that annoying request and get the images we get a list of volumes and pipe them to the _Get-DiskImage_ cmdlet:

```
PS> Get-Volume | Get-DiskImage
Attached          : True
BlockSize         : 0
DevicePath        : \\.\CDROM1
FileSize          : 3618824192
ImagePath         : D:\ISO\en_windows_8_ent_x64.iso
LogicalSectorSize : 2048
Number            : 1
Size              : 3618824192
StorageType       : 1
PSComputerName    :

Attached          : True
BlockSize         : 0
DevicePath        : \\.\CDROM2
FileSize          : 3490912256
ImagePath         : D:\ISO\mu_exchange_server_2013_x64.iso
LogicalSectorSize : 2048
Number            : 2
Size              : 3490912256
StorageType       : 1
PSComputerName    :
```

If you take a second look at the output of each command you will notice that the result doesn&#8217;t reflect the action of the command. The output shows the state of the image object **before** it has been mounted or dismounted. There&#8217;s an [open bug report on Connect][1] if you want to vote on it.

Finally, using that trick we can quickly dismount all mounted (iso based) drives:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Volume | Get-DiskImage | Dismount-DiskImage
</pre>

[1]: https://connect.microsoft.com/WindowsServer/feedback/details/773945/mount-diskimage-passthru-outputs-the-image-before-its-mounted