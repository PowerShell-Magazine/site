---
title: '#PSCXTip Manipulating file attributes'
author: Keith Hill
type: post
date: 2012-12-20T19:00:18+00:00
url: /2012/12/20/pscxtip-manipulating-file-attributes/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When dealing with files under source control it is quite often useful to be able to make a bunch of files readonly or writable in bulk. You can do this fairly easily in PowerShell 3.0:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-ChildItem . –Recurse –File | Foreach {$_.IsReadOnly = $false}
</pre>

However the <a title="PowerShell Community Extensions" href="http://pscx.codeplex.com" target="_blank">PowerShell Community Extensions</a> (1.2 or higher) provides an even easier way to do this common operation with two commands _Set-ReadOnly_ (alias sro) and _Set-Writable_ (alias swr). The above operation simplifies to this:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-ChildItem . –Recurse –File | Set-ReadOnly
</pre>

If you only need to make a set of files in the current directory readonly or writable, you can use this form:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Set-Writable *.cs
</pre>

Or using the aliases:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; swr *.cs; sro *.csproj
</pre>

One final common file manipulation is to “touch” a file to set its last write time to the current time (or any specified time). This can be done with the PSCX command _Set-FileTime_ (alias touch) e.g.:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Set-FileTime *.cs
</pre>

Or using the alias:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; touch *.cs
</pre>

If you need to set the last write time to a specific time you can use the _Time_ parameter e.g.:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-ChildItem . –r –File | Set-FileTime -Time ((Get-Date).AddDays(-14))
</pre>

By default _Set-FileTime_ updates the last write and last accessed times. You can specify the _–Created_ switch to update the file’s creation time.

**Note**: There are many more useful PowerShell Community Extensions (PSCX) commands. If you are interested in this great community project led by PowerShell MVPs <a title="Keith Hill's blog" href="http://rkeithhill.wordpress.com" target="_blank">Keith Hill</a> and <a title="Oisin Grehan's blog" href="http://www.nivot.org" target="_blank">Oisin Grehan</a>, give PSCX a try at <http://pscx.codeplex.com>.