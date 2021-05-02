---
title: '#PSTip How can I determine if I’m in debugging mode?'
author: Shay Levy
type: post
date: 2013-01-10T19:00:34+00:00
url: /2013/01/10/pstip-how-can-i-determine-if-im-in-debugging-mode/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

When a session is being debugged, PowerShell populates an automatic variable called _$PSDebugContext _which contains an object that has Breakpoints and InvocationInfo properties information about the debugging environment. Visually you can see this by looking at the prompt. In debugging mode, &#8220;[DBG]&#8221; is added to the prompt:

<pre class="brush: powershell; title: ; notranslate" title="">[DBG] PS C:\&gt;
</pre>

If you need to detect this programatically, you can use the _Test-Path_ cmdlet and check for the existence of the variable . If _$PSDebugContext _exists (not $null),  you are in debugging mode.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Test-Path Variable:PSDebugContext
</pre>