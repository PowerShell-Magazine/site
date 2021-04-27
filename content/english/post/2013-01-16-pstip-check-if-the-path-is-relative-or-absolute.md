---
title: '#PSTip Check if the path is relative or absolute'
author: Ravikanth C
type: post
date: 2013-01-16T19:00:33+00:00
url: /2013/01/16/pstip-check-if-the-path-is-relative-or-absolute/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

You may want to check if a provided file/folder path is relative or absolute and perform appropriate action based on the result. The <a href="http://msdn.microsoft.com/en-us/library/3bdzys9w.aspx" target="_blank">System.IO.Path</a> namespace provides the <a href="http://msdn.microsoft.com/en-us/library/system.io.path.ispathrooted.aspx" target="_blank">IsPathRooted()</a> static method. This method will return True if the path is absolute and False if it is relative. The usage of this is quite simple.

<pre class="brush: powershell; title: ; notranslate" title="">[System.IO.Path]::IsPathRooted("../Scripts")
</pre>

This will result in $false as the path we provided is a relative path.

This is useful especially when we are validating function parameters. You can just use this as a part of ValidateScript attribute:


    function Test-AbsolutePath {
        Param (
            [Parameter(Mandatory=$True)]
            [ValidateScript({[System.IO.Path]::IsPathRooted($_)})]
            [String]$Path
        )
        #....
        # Your script logic here
    }
When you use the ValidateScript attribute in the function parameters, the value of -Path is first validated to check if the path is absolute or not before running the actual function code.