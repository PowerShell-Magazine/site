---
title: '#PSTip Get the size of a file on disk'
author: Ravikanth C
type: post
date: 2013-10-31T18:00:22+00:00
url: /2013/10/31/pstip-get-the-size-of-a-file-on-disk/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier post, I wrote about [compressing / uncompressing files using WMI][1]. Towards the end of that article, the image showed the difference in file size on disk before and after the compression. So, how can we retrieve that value using PowerShell? Let us look at _Get-Item_ and WMI to see if we can retrieve the file size on disk.

![](/images/compress.png)

    namespace Disk
    {
        public class SizeInfo
        {
            [DllImport("kernel32.dll", SetLastError=true, EntryPoint="GetCompressedFileSize")]
            static extern uint GetCompressedFileSizeAPI(string lpFileName, out uint lpFileSizeHigh);
    	
            public static ulong GetCompressedFlieSize(string FileName,bool SizeInMB)
            {
                uint HighOrder;
                uint LowOrder;
                LowOrder = GetCompressedFileSizeAPI(FileName, out HighOrder);
                int error = Marshal.GetLastWin32Error();
                if (HighOrder == 0 && LowOrder == 0xFFFFFFFF && error != 0)
                    throw new Win32Exception(error);
                else
                    if (SizeInMB)
                        return ((((ulong)HighOrder &lt;&lt; 32) + LowOrder)/1024)/1024;
                    else
                        return ((ulong)HighOrder &lt;&lt; 32) + LowOrder;
            }
    	}
    }
    `@
The above snippet will create the _Disk.SizeInfo_ namespace in the current PowerShell session. We can use this namespace to retrieve the file size on disk.

```
[Disk.SizeInfo]::GetCompressedFlieSize( "C:\scripts\test.vhd",$false)
[Disk.SizeInfo]::GetCompressedFlieSize( "C:\scripts\test.vhd",$true)
```


If the second parameter is set to $true, the value of the file size will be returned in megabytes.

[1]: /2013/10/28/pstip-compress-and-uncompress-files-and-folders-using-wmi/