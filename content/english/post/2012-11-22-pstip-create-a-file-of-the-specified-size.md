---
title: '#PSTip Create a file of the specified size'
author: Shay Levy
type: post
date: 2012-11-22T19:00:18+00:00
url: /2012/11/22/pstip-create-a-file-of-the-specified-size/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Sometimes you need to create a file of the specified size, as a placeholder for instance. There are many utilities that do that (e.g. <a title="fsutil" href="http://technet.microsoft.com/en-us/library/cc788058(v=ws.10).aspx" target="_blank">fsutil</a> ) but in this tip I&#8217;ll show you how to create a file of the specified size using a .NET class.

```
function New-EmptyFile
{
   param( [string]$FilePath,[double]$Size )

   $file = [System.IO.File]::Create($FilePath)
   $file.SetLength($Size)
   $file.Close()
   Get-Item $file.Name
}
```

For example, you can use the New-EmptyFile function to create a 20 MB file:

```
PS> New-EmptyFile -FilePath c:\temp\test.txt -Size 20mb
	Directory: C:\temp

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        11/22/2012   3:39 PM   20971520 test.txt
```