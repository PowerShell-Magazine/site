---
title: '#PSTip Generate a zero-byte, temporary file on disk'
author: Ravikanth C
type: post
date: 2012-12-04T19:00:42+00:00
url: /2012/12/04/pstip-generate-a-zero-byte-temporary-file-on-disk/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 1.0 or above.

In an earlier tip, we showed you how to [create a file of the specified size][1]. In today&#8217;s post, let us see how we can use another .NET class &#8211; System.IO.Path &#8211; to create a temporary, zero-byte file on the disk.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [System.IO.Path]::GetTempFileName()
C:\Users\ravi\AppData\Local\Temp\tmp2D48.tmp
</pre>

The [GetTempFileName()][2] method can be quite useful when you need to generate a random temporary file for writing log information from a script or for saving state of your script while the execution is in progress. As shown above, this method returns full path of the newly created temporary file and always creates temporary files with an extension .TMP.

[1]: /2012/11/22/pstip-create-a-file-of-the-specified-size/
[2]: http://msdn.microsoft.com/en-us/library/system.io.path.gettempfilename(v=vs.90).aspx