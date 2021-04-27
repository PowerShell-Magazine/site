---
title: '#PSTip Detecting if the console is in Interactive mode'
author: Shay Levy
type: post
date: 2013-05-13T18:00:00+00:00
url: /2013/05/13/pstip-detecting-if-the-console-is-in-interactive-mode/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
So, in your scripts you want to gather information from the user who runs it and you use the _Read-Host_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Read-Host Please enter your name
</pre>

This of course works most of time but there are cases where it won&#8217;t work. One case is when the console has been launched using the NonInteractive switch. When that happens you will get this error:

<pre class="brush: powershell; title: ; notranslate" title="">Read-Host : Windows PowerShell is in NonInteractive mode. Read and Prompt functionality is not available.
</pre>

How can you tell if the console allows user interaction? One way to avoid that is to detect whether the host that runs the script has been launched in non-interactive mode.  You can find the switches and arguments used to launch your console using the [_Environment.GetCommandLineArgs_][1] method.  With the  _GetCommandLineArgs_ method you can get the array of the command-line arguments for the current process.

<pre class="brush: powershell; title: ; notranslate" title="">C:\&gt; powershell -NoProfile -NoLogo -NonInteractive -Command "[Environment]::GetCommandLineArgs()"
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
-NoProfile
-NoLogo
-NonInteractive
-Command
[Environment]::GetCommandLineArgs()
</pre>

The array includes the executable path along with the command-line arguments used to invoke it (if any). Now we can use that list to check if one of the arguments starts with ‘-noni’ . We check for ‘-noni*’ because there are two ways to specify the the NonInteractive switch, using it’s full name, or using a its short version.  Using a wildcard pattern covers both cases. We then cast the result to a Boolean so we can have a True/False result.

**Note** that this will work only for console based hosts (powershell.exe), not in the ISE.

<pre class="brush: powershell; title: ; notranslate" title="">[bool]([Environment]::GetCommandLineArgs() -like '-noni*')
</pre>

[1]: http://msdn.microsoft.com/en-us/library/system.environment.getcommandlineargs.aspx