---
title: '#PSTip How to get non-running automatic services'
author: Aleksandar Nikolic
type: post
date: 2012-08-17T18:00:07+00:00
url: /2012/08/17/pstip-how-to-get-non-running-automatic-services/
views:
  - 7473
post_views_count:
  - 1362
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The Startup type information is not available when we use the Get-Service cmdlet. Luckily the Win32_Service WMI class has a StartMode property that we can use to create a nice server-side filter. If outer quotes are double quotes, you need to use single ones for values.

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Class Win32_Service -Filter "StartMode='Auto' AND State='Stopped'"
</pre>

You can add ComputerName parameter to easily query services on a remote computer:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Class Win32_Service -Filter "StartMode='Auto' AND State='Stopped'" –ComputerName Server01,Server02 –Credential (Get-Credential Administrator)
</pre>

You cannot use the Credential parameter when you run the management operation against the local computer.