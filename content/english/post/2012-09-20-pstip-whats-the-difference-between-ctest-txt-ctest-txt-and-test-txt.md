---
title: '#PSTip What’s the difference between ${c:\test.txt}, ${c:test.txt}, and ${test.txt}'
author: Aleksandar Nikolic
type: post
date: 2012-09-20T18:00:58+00:00
url: /2012/09/20/pstip-whats-the-difference-between-ctest-txt-ctest-txt-and-test-txt/
views:
  - 5972
post_views_count:
  - 1069
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Each PowerShell drive (PSDrive) has its own notion of current working directory (CurrentLocation):

```
PS C:\> Get-PSDrive d,c
Name Used (GB) Free (GB) Provider   Root CurrentLocation
---- --------- --------- --------   ---- ---------------
D       295.59    102.49 FileSystem D:\
C       277.61    205.11 FileSystem C:\             temp
```

In the variable notation, if the path after the drive specifier is a relative path, then it will be resolved relative to the current working directory for the specified drive. If the drive is not specified, then the current directory for the current drive is used.

  * ${c:\test.txt} refers to the file in the root of the C: drive
  * ${c:test.txt} refers to the file in the current working directory on the C: drive
  * ${test.txt} refers to the file in the current working directory on the current drive

This works for commands as well:

```
# look up test.txt file in the root of the C: drive
PS C:\temp> Get-Content C:\test.txt

# look up test.txt file relative to the current directory for the C: drive
# in this case that's C:\temp\test.txt
PS C:\temp> Get-Content C:test.txt

# look up test.txt relative to the current directory on the current drive
PS C:\temp> cd D:
PS D:\>

# in this case that's D:\test.txt
PS D:\> Get-Content test.txt
```