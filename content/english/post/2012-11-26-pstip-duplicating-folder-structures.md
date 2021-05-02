---
title: '#PSTip Duplicating folder structures'
author: Shay Levy
type: post
date: 2012-11-26T19:00:58+00:00
url: /2012/11/26/pstip-duplicating-folder-structures/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Sometimes you need to create a copy of an exiting directory structure without copying the files, and you also want to include empty folders if they exist. An example would be a deep project directory structure that needs to be created for a new project.

In the days of DOS you could use the XCOPY command. The following command creates the Windows directory structure under the temp directory of drive D:.

<pre class="brush: powershell; title: ; notranslate" title="">XCOPY c:\Windows d:\temp\Windows /E /T /I
</pre>

In PowerShell you can use the Copy-Item cmdlet. The Filter scriptblock specifies the PSIsContainer property which passes on just folder objects:

<pre class="brush: powershell; title: ; notranslate" title="">Copy-Item $env:windir d:\temp\windows -Filter {PSIsContainer} -Recurse -Force
</pre>