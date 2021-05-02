---
title: '#PSTip Clear clipboard content'
author: Ravikanth C
type: post
date: 2013-11-13T19:00:08+00:00
url: /2013/11/13/pstip-clear-clipboard-content/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Have you ever wondered how to clear the contents of the clipboard? There are certainly many ways to do that in PowerShell. Let us see a couple of them.

<pre class="brush: powershell; title: ; notranslate" title="">echo $null | clip.exe
</pre>

The above snippet is the most trivial. Then, we can use the .NET way of doing it.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName System.Windows.Forms
[System.Windows.Forms.Clipboard]::Clear()
</pre>

The above snippet needs the Windows Forms namespace to clear the clipboard. Note that you don&#8217;t need to add the assembly if you are running the above snippet in PowerShell ISE. We can also use the native Win32 API &#8211; [EmptyClipboard][1]. This is slightly complex and requires either Pinvoke or .NET Interop.

[1]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms649037(v=vs.85).aspx