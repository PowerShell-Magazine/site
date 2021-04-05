---
title: '#PSTip Compress files and folders with System.IO.Compression.FileSystem class'
author: Jaap Brasser
type: post
date: 2015-06-09T18:00:37+00:00
url: /2015/06/09/pstip-compress-files-and-folders-with-system-io-compression-filesystem-class/
views:
  - 19255
post_views_count:
  - 4458
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Using the CreateFromDirectory and ExtractToDirectory methods, it is possible to compress and extract files. In this tip I will show different constructors that can be used to either compress or extract files using this class. The following example will compress the files stored in the c:\testing folder:

```powershell
Add-Type -Assembly 'System.IO.Compression.FileSystem'
[System.IO.Compression.ZipFile]::CreateFromDirectory('c:\testing', 'c:\testing.zip','Optimal',$false)
```


When you want to extract files, use the ExtractToDirectory method:

```powershell
[System.IO.Compression.ZipFile]::ExtractToDirectory('c:\testing.zip', 'c:\newtest')
```


In PowerShell 5.0 the Compress-Archive and Expand-Archive cmdlets are available to simplify working with compressed archives.

For more information about the ZipFile class and the Compress-Archive and Expand-Archive cmdlets please refer to the following articles:

[ZipFile Class][1]

[Compress-Archive][2]

[Expand-Archive][3]

[1]: https://msdn.microsoft.com/en-us/library/system.io.compression.zipfile
[2]: https://technet.microsoft.com/en-us/library/dn841358.aspx
[3]: https://technet.microsoft.com/en-us/library/dn841359.aspx