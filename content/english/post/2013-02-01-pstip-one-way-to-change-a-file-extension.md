---
title: '#PSTip One way to change a file extension'
author: Jakub Jareš
type: post
date: 2013-02-01T19:00:44+00:00
url: /2013/02/01/pstip-one-way-to-change-a-file-extension/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Modifying a file always brings a risk of corrupting it; making a backup copy whenever possible is one of the file manipulation best-practices. One way of doing it is to create a copy with the same name and appropriate extension like &#8216;bak&#8217;. This task can be finished in a few different ways. Here is my favorite&#8211;using the _Copy-Item_ cmdlet and the _ChangeExtension()_ method of the _System.IO.Path_ class:

<pre class="brush: powershell; title: ; notranslate" title="">$path = 'C:\temp\ImportantFile.txt'
Copy-Item -Path $path –Destination ([io.path]::ChangeExtension($path, '.bak')) -Verbose
</pre>