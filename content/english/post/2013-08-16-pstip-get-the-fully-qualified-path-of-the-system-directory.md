---
title: '#PSTip Get the fully qualified path of the system directory'
author: Shay Levy
type: post
date: 2013-08-16T18:00:20+00:00
url: /2013/08/16/pstip-get-the-fully-qualified-path-of-the-system-directory/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
There are many ways to get the path of the system directory (e.g system32). Here&#8217; one that I find very useful that involves the .NET Environment class:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [System.Environment]::SystemDirectory
C:\WINDOWS\system32
</pre>

I use it in my PowerShell profile and adds it to my session&#8217;s _Env_ drive so I can conveniently query its value  just as I query all environment variables:

```
# add a new environment variable
PS> $env:SystemDirectory = [Environment]::SystemDirectory

# query it using the $env drive variable
PS> $env:SystemDirectory
C:\WINDOWS\system32
```

Which also makes it very easy to be embedded and expanded in a string:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; "The path of the system directory is: $env:SystemDirectory"
The path of the system directory is: C:\WINDOWS\system32
</pre>