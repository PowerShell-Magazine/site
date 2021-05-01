---
title: '#PSTip Compress and uncompress files and folders using WMI'
author: Ravikanth C
type: post
date: 2013-10-28T18:00:55+00:00
url: /2013/10/28/pstip-compress-and-uncompress-files-and-folders-using-wmi/
categories:
  - WMI
  - Tips and Tricks
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 2.0 or above.

In WMI, the _[CIM_DataFile][1]_ and _[CIM_Directory][2]_ classes have some very useful methods we can use. For example, the _[Compress()][3]_ and the _[Uncompress()][4]_ methods can be used to reduce the on-disk footprint of a file or a folder. Essentially, think of this as a poor man&#8217;s compression toolkit! These methods use NTFS compression.

In today&#8217;s tip, I will show you how you can use these methods to work with file/folder compression. There are multiple ways we can invoke WMI methods in PowerShell. For the purpose of this tip, I will stick to calling the methods on a WMI object using the dotted notation.

<pre class="brush: powershell; title: ; notranslate" title="">$file = Get-WmiObject -Query "SELECT * FROM CIM_DataFile WHERE Name='C:\\scripts\\test.vhd'"
$file.Compress()
</pre>

The above code snippet compresses the file. To uncompress, we simply use the the _Uncompress()_ method.

<pre class="brush: powershell; title: ; notranslate" title="">$file = Get-WmiObject -Query "SELECT * FROM CIM_DataFile WHERE Name='C:\\scripts\\test.vhd'"
$file.Uncompress()
</pre>

The _Compressed_ property of _CIM_DataFile_ instance tells us if the file is compressed or not.

To see if a file is really compressed or not, we can verify its size on the disk.

![](/images/filecompression.png)

Now, how do we compress a folder? Simple. We use the Compress() method of a CIM_Directory instance.

```
$folder = Get-WmiObject -Query "SELECT * FROM CIM_Directory WHERE Name='C:\\scripts'"
$folder.Compress()
```


[1]: http://msdn.microsoft.com/en-us/library/aa387236(v=vs.85).aspx
[2]: http://msdn.microsoft.com/en-us/library/gg196431(v=vs.85).aspx
[3]: http://msdn.microsoft.com/en-us/library/aa389255(v=vs.85).aspx
[4]: http://msdn.microsoft.com/en-us/library/aa393932(v=vs.85).aspx