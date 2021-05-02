---
title: '#PSTip Finding the drive letter of a mounted disk image'
author: Ravikanth C
type: post
date: 2013-03-07T19:00:03+00:00
url: /2013/03/07/pstip-finding-the-drive-letter-of-a-mounted-disk-image/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Windows 8 and Windows Server 2012 included cmdlets to manage storage. One such cmdlet is _Mount-DiskImage_ which can be used to mount ISO, VHD, and VHDX files. But, unfortunately, this cmdlet does not give us information on the drive letter assigned to the newly mounted disk image. Knowing the drive letter might be a very important thing when using these cmdlets in a script.

So, how do we find the drive letter? Here is one method:

<pre class="brush: powershell; title: ; notranslate" title="">$before = (Get-Volume).DriveLetter
Mount-DiskImage -ImagePath C:\test\Test.vhd -StorageType VHD
$after = (Get-Volume).DriveLetter
compare $before $after -Passthru
</pre>
![](/images/drive.png)

There are other methods too&#8211;such as using PowerShell eventing or background jobs, etc. Let us save that for future tips! 🙂