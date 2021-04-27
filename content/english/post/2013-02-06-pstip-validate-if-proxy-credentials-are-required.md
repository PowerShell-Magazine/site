---
title: '#PSTip Validate if proxy credentials are required'
author: Ravikanth C
type: post
date: 2013-02-06T19:01:15+00:00
url: /2013/02/06/pstip-validate-if-proxy-credentials-are-required/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I write scripts that include downloading content from the Internet. Now, I want these scripts to work even when I am behind a proxy server. There is a simple way in .NET to check if we are behind a proxy. For this purpose, we will use System.Net.WebClient .NET class.

<pre class="brush: powershell; title: ; notranslate" title="">Function Test-Proxy {
 $wc = New-Object System.Net.WebClient
 $wc.Proxy.IsBypassed("http://www.microsoft.com")
}
</pre>

The _Test-Proxy_ function returns _False_ if you are behind a proxy server.

**Note:** The following example works only on PowerShell 3.0 and above.

I use this effectively along with the [_$PSDefaultParameterValues_][1] in PowerShell 3.0. For example, in a script to download content from Internet, I will have to set proxy credentials for the _Start-BitsTransfer_ cmdlet. This is how I do it:

<pre class="brush: powershell; title: ; notranslate" title="">if (-not (Test-Proxy)) {
    if (-not ($PSDefaultParameterValues.ContainsKey("Start-BitsTransfer:ProxyAuthentication")) -and ($PSDefaultParameterValues.ContainsKey("Start-BitsTransfer:ProxyAuthentication"))) {
        $cred = Get-Credential -Message "Enter Proxy authentication credentials" -UserName "${env:USERDOMAIN}\${env:UserName}"
        $PSDefaultParameterValues.Add("Start-BitsTransfer:ProxyAuthentication","Basic")
        $PSDefaultParameterValues.Add("Start-BitsTransfer:ProxyCredential",$cred)
    }
}
</pre>

If we place the above code snippet at the beginning of the script, it first checks if proxy authentication is required for Internet access and sets the default parameter values for _Start-BitsTransfer_ cmdlet.

[1]: http://technet.microsoft.com/en-us/library/hh847819.aspx