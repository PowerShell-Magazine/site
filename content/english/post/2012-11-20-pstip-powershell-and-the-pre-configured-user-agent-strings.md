---
title: '#PSTip PowerShell and the pre-configured user agent strings'
author: Aleksandar Nikolic
type: post
date: 2012-11-20T23:23:49+00:00
url: /2012/11/20/pstip-powershell-and-the-pre-configured-user-agent-strings/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 3.0 or above.

The _Invoke-WebRequest_ and _Invoke-RestMethod_ cmdlets have the _-UserAgent_ parameter, so you can specify a user agent string for the web request. A user agent string has &#8220;Compatibility (Platform; OS; Culture) App&#8221; format, and by default, PowerShell 3.0 identifies itself as &#8220;Mozilla/5.0 (Windows NT; Windows NT 6.1; en-US) WindowsPowerShell/3.0&#8221; on my Windows 7 machine. What values should we specify if we want to customize the user agent string? Static properties of the _[Microsoft.PowerShell.Commands.PSUserAgent]_ class provide some pre-configured values:

```
PS> [Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() |
Select-Object Name, @{n='UserAgent';e={ [Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name) }}
Name             UserAgent
----             ---------
InternetExplorer Mozilla/5.0 (compatible; MSIE 9.0; Windows NT; Windows NT 6.1; en-US)
FireFox          Mozilla/5.0 (Windows NT; Windows NT 6.1; en-US) Gecko/20100401 Firefox/4.0
Chrome           Mozilla/5.0 (Windows NT; Windows NT 6.1; en-US) AppleWebKit/534.6 (KHTML, like Gecko) Chrome/7.0.500.0 Safari/534.6
Opera            Opera/9.70 (Windows NT; Windows NT 6.1; en-US) Presto/2.2.1
Safari           Mozilla/5.0 (Windows NT; Windows NT 6.1; en-US) AppleWebKit/533.16 (KHTML, like Gecko) Version/5.0 Safari/533.16
```

We can use it like in the following command:

<pre class="brush: powershell; title: ; notranslate" title="">PS> $userAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
PS> Invoke-WebRequest https://powershellmagazine.com -UserAgent $userAgent
</pre>