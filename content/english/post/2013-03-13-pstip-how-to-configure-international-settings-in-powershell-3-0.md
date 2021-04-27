---
title: '#PSTip How to configure International settings in PowerShell 3.0'
author: Shay Levy
type: post
date: 2013-03-13T18:00:41+00:00
url: /2013/03/13/pstip-how-to-configure-international-settings-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Windows 8 and Windows Server 2012 ships with a new module worth exploring: the _International_ module. You can use the module cmdlets to control the language that is used for various elements of the user interface (UI). Since the early days of PowerShell, we could get the current culture with the _Get-Culture_ cmdlet but we couldn&#8217;t set it. The _International_ module enables us to set the culture with the _Set-Culture_ cmdlet.

The _Set-Culture_ cmdlet enables you to quickly set the user culture for the current user account. The information includes the names for the culture, the writing system, the calendar, and formatting for dates and sort strings.

<pre class="brush: powershell; title: ; notranslate" title="">Set-Culture -CultureInfo he-IL
</pre>

Here&#8217;s a list of the _International_ module commands:

```
PS> Get-Command -Module International
CommandType     Name                                               ModuleName
-----------     ----                                               ----------
Cmdlet          Get-WinAcceptLanguageFromLanguageListOptOut        International
Cmdlet          Get-WinCultureFromLanguageListOptOut               International
Cmdlet          Get-WinDefaultInputMethodOverride                  International
Cmdlet          Get-WinHomeLocation                                International
Cmdlet          Get-WinLanguageBarOption                           International
Cmdlet          Get-WinSystemLocale                                International
Cmdlet          Get-WinUILanguageOverride                          International
Cmdlet          Get-WinUserLanguageList                            International
Cmdlet          New-WinUserLanguageList                            International
Cmdlet          Set-Culture                                        International
Cmdlet          Set-WinAcceptLanguageFromLanguageListOptOut        International
Cmdlet          Set-WinCultureFromLanguageListOptOut               International
Cmdlet          Set-WinDefaultInputMethodOverride                  International
Cmdlet          Set-WinHomeLocation                                International
Cmdlet          Set-WinLanguageBarOption                           International
Cmdlet          Set-WinSystemLocale                                International
Cmdlet          Set-WinUILanguageOverride                          International
Cmdlet          Set-WinUserLanguageList                            International
```