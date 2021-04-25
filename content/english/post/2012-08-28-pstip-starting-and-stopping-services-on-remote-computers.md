---
title: '#PSTip Starting and Stopping services on remote computers'
author: Shay Levy
type: post
date: 2012-08-28T18:00:47+00:00
url: /2012/08/28/pstip-starting-and-stopping-services-on-remote-computers/
views:
  - 24980
post_views_count:
  - 2516
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Consider the following PowerShell 2.0 command:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Service -Name wuauserv -ComputerName Server1 | Stop-Service
</pre>

What do you think is happening here? You get the Windows Update service from Server1 and stop it, right? Wrong!

There&#8217;s a bug in PowerShell 2.0 and what actually happens is that Stop-Service stops the service on the local computer.

This has ben fixed in v3 but there&#8217;s a trick you can use to make it work in v2, pass the service instance to the InputObject Parameter:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $svc = Get-Service -Name wuauserv -ComputerName Server1
PS&gt; Stop-Service -InputObject $svc
</pre>