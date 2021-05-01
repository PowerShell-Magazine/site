---
title: '#PSTip Setting keyboard layout in Windows 8 and Server 2012'
author: Ravikanth C
type: post
date: 2014-02-06T19:00:23+00:00
url: /2014/02/06/pstip-setting-keyboard-layout-in-windows-8-and-server-2012/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Have you ever wondered how you can set the keyboard layout using PowerShell? I recently came across a situation where a set of virtual machines I deployed from a template had a different keyboard layout than what I intend to use. Fortunately, starting with Windows Server 2012 and Windows 8, there is a built-in cmdlet to do this.

The [Set-WinUserLanguageList][1] cmdlet can be used to set the keyboard layout. This cmdlet is available in the [International][2] module.

<pre class="brush: powershell; title: ; notranslate" title="">Set-WinUserLanguageList -LanguageList en-US
</pre>

The Get-WinUserLanguageList cmdlet gets the list of languages for the current user.

[1]: http://technet.microsoft.com/en-us/library/hh852168.aspx
[2]: http://104.131.21.239/2013/03/13/pstip-how-to-configure-international-settings-in-powershell-3-0/