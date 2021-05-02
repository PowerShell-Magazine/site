---
title: '#PSTip Capture console screen'
author: Shay Levy
type: post
date: 2012-11-19T19:00:31+00:00
url: /2012/11/19/pstip-capture-console-screen/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

PowerShell has many logging capabilities but sometimes what you want is to capture what&#8217;s already written to the console. Capturing this information can be a challenge as you need to script the console buffer content. You can see how in [this post by the PowerShell team blog][1]. In the ISE however this task becomes a lot easier; it is just a matter of reading the content of the output (console) pane.

In ISE v2:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $psise.CurrentPowerShellTab.Output.Text
</pre>

In ISE v3:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $psise.CurrentPowerShellTab.ConsolePane.Text
</pre>

[1]: http://blogs.msdn.com/b/powershell/archive/2009/01/10/capture-console-screen.aspx