---
title: '#PSTip Determine compression ratio of compressed files'
author: Jaap Brasser
type: post
date: 2015-06-10T18:00:55+00:00
url: /2015/06/10/pstip-determine-compression-ratio-of-compressed-files/
views:
  - 9457
post_views_count:
  - 2414
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
In the previous <a href="/2015/06/09/pstip-compress-files-and-folders-with-system-io-compression-filesystem-class/" target="_blank">#PSTip Compress files and folders with System.IO.Compression.FileSystem class</a>, a .zip file was created and extracted. Now it might be good to know the compression ratio to see how much storage space has been saved by compressing the files. Using a ForEach-Object loop in combination with the Open method of this class we can determine the total size, compressed size, and the ratio of compression of a .zip file.

```powershell
[System.IO.Compression.ZipFile]::Open("c:\testing\colorcopy\yourfile.zip",'Read') | ForEach-Object {

     $_.Entries | ForEach-Object -Begin {
        [long]$TotalCompressed = $null
        [long]$TotalSize = $null
    } -Process {
        $TotalCompressed += $_.CompressedLength
        $TotalSize += $_.Length
    } -End {
        [pscustomobject]@{
            FileSize = $TotalSize
            CompressedSize = $TotalCompressed
            Ratio = "{0:P2}" -f ($TotalCompressed / $TotalSize)
        }
    }
} 
```

