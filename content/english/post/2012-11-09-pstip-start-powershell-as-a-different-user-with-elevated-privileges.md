---
title: '#PSTip Start PowerShell as a different user with elevated privileges'
author: Aleksandar Nikolic
type: post
date: 2012-11-09T18:51:29+00:00
url: /2012/11/09/pstip-start-powershell-as-a-different-user-with-elevated-privileges/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

This command is useful when you want to perform some tests or if you work in an environment where you use the standard and admin user accounts. Open Command Prompt and run the following command:

<pre class="brush: powershell; title: ; notranslate" title="">C:\&gt; C:\Windows\System32\runas.exe /env /noprofile /user:CHANGEME@TEST.LOCAL "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -noprofile -command \"start-process powershell -verb RunAs\""
</pre>

The best way to use it is to create a shorcut on your desktop and put it in the Target field.