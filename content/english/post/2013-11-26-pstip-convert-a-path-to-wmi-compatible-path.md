---
title: '#PSTip Convert a path to WMI compatible path'
author: Ravikanth C
type: post
date: 2013-11-26T19:00:44+00:00
url: /2013/11/26/pstip-convert-a-path-to-wmi-compatible-path/
categories:
  - WMI
  - Tips and Tricks
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 3.0 or above.

Some of the WMI queries require that we specify the folder path with an escaped path separator. For example, take a look at the following error.

![](/images/cimerror.png)

So, we need to escape the path separator for the above query to work.

![](/images/cimnoerror.png)

When we are running such queries programmatically for user provided input, we need a better way to convert the path to WMI compatible path.

Here is the trick I use.

<pre class="brush: powershell; title: ; notranslate" title="">'C:\Windows' -replace "\\","\\"
</pre>

How do you do it?