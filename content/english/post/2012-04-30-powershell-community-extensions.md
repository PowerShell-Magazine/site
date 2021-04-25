---
title: PowerShell Community Extensions
author: Keith Hill
type: post
date: 2012-04-30T18:00:25+00:00
url: /2012/04/30/powershell-community-extensions/
views:
  - 16392
post_views_count:
  - 1839
categories:
  - PSCX
  - Module Spotlight
tags:
  - Modules
  - PSCX

---
I started the PowerShell Community Extensions (PSCX) project to fill in  some of the gaps in the set of built-in Windows PowerShell commands.  Over the years, some of the PSCX command equivalents have made their way into Windows PowerShell including Start-Process, Select-Xml, Get-WebService_,_ and Get-Random.  However, for anyone who has used the standard set of UNIX utilities via packages such as cygwin or MKS Toolkit knows, there are still quite a few missing commands in Windows PowerShell.  Here are some of my favorites that PSCX provides.

One of the first cmdlets I missed was a PowerShell equivalent to the octal dump (od) utility.  In PowerShell terms, we would call this a formatter as in Format-Hex:

```powershell
PS> [byte[]](1..32) | Set-Content -Encoding Byte foo.bin
PS> Format-Hex foo.bin –HideAscii

Address:  0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
-------- -----------------------------------------------
00000000 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10
00000010 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F 20
```

Another PSCX formatter I use a lot is the Format-Xml cmdlet.  This cmdlet is great for viewing pesky single-line, 8000 character wide XML files:

<pre>PS&gt; $xml = '&lt;doc&gt;&lt;title name="foo" desc="bar"/&gt;&lt;/doc&gt;'
PS&gt; $xml | Format-Xml –AttributesOnNewLine
&lt;doc&gt;
&lt;title
name="foo"
desc="bar" /&gt;
&lt;/doc&gt;</pre>

As a developer, I deal with lots of XML files (app.config files, MSBuild and TeamBuild project files, etc).  One cmdlet I find very handy to use on these XML files before checking them in is Test-Xml:

<pre>PS&gt; '&lt;doc&gt;&lt;title&gt;&lt;/doc&gt;' | Test-Xml –Verbose
VERBOSE: The 'title' start tag on line 1 does not match the
end tag of 'doc'. Line 1, position 15.
False</pre>

The Test-Xml cmdlet verifies that the XML is well-formed and if a schema is provided it will also validate the XML against the schema.

I’ve contributed a number of PowerShell scripts to our build and test processes.  Those PowerShell scripts can break our build if I’m not careful when I’m checking in updates.  I use the Test-Script cmdlet to make sure a script is free of syntax errors before checking it in:

<pre>PS&gt; 'for($i=0;$i&lt;10;$i++){$i}' | Test-Script
WARNING: Parse error on line:1 char:12 - The '&lt;' operator
is reserved for future use.
False</pre>

Note: the Test-Script cmdlet uses the _PSParser tokenizer_ provided in PowerShell v2 which only catches syntax errors and not runtime errors.

If you’ve ever needed to execute a batch file to modify environment variables for the current PowerShell session, then Invoke-BatchFile will come in very handy.  Normally with the execution of a batch file, the new environment variable definitions exist only in the spawned cmd.exe process.  Invoke-BatchFile will import those environment variable definitions back into the PowerShell session that executed the batch file.  I use this command to import Visual Studio environment variables into my PowerShell session:

<pre>PS&gt; Invoke-BatchFile "${env:VS100COMNTOOLS}..\..\VC\vcvarsall.bat" amd64</pre>

For developers, the Test-Assembly cmdlet can be useful in your build scripts.  For example, when you need to re-sign partially signed assemblies you don’t want to apply the re-signing utility to native DLLs:

<pre>PS&gt; Get-ChildItem *.dll | where {Test-Assembly $_} |
&gt;&gt;    foreach {sn.exe -R $_ key.snk}
&gt;&gt;</pre>

One of the commands that can be useful to any PowerSheller is a native application called echoargs.exe. You use it when you’re trying to troubleshoot problems passing arguments to native applications.  You stand a much better chance of fixing the problem if you can see what PowerShell is passing to the native application.  echoargs.exe is used as a stand-in for your application.  All it does is showing you the command line arguments that PowerShell passes to the application.  This Team Foundation command fails when executed in PoweShell: tf.exe status . /r /workspace:*;hillr.  If you substitute echoargs for the .exe file, then you can see what is going on:

<pre>PS&gt; echoargs status . /r /workspace:*;hillr
Arg 0 is &lt;status&gt;
Arg 1 is &lt;.&gt;
Arg 2 is &lt;/r&gt;
Arg 3 is &lt;/workspace:*&gt;</pre>

The term hillr is not recognized as the name of a cmdlet.

echoargs shows that the ;hillr part of the argument doesn’t even make it to the application.  The problem is that the “;” character is a statement separator in PowerShell.  You need to put quotes around the argument to ensure it gets to tf.exe in one piece:

<pre>PS&gt; echoargs status . /r '/workspace:*;hillr'
Arg 0 is &lt;status&gt;
Arg 1 is &lt;.&gt;
Arg 2 is &lt;/r&gt;
Arg 3 is &lt;/workspace:*;hillr&gt;</pre>

There are many more useful PSCX commands such as Set-FileTime, Set-Writable, Set-ReadOnly, Unblock-File, and Show-Tree.  Hopefully I’ve piqued your interest in the PowerShell Community Extensions.  If so, give PSCX a try at <http://pscx.codeplex.com>.