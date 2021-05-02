---
title: '#PSTip Retrieve a redirected URL'
author: Ravikanth C
type: post
date: 2013-01-29T19:01:02+00:00
url: /2013/01/29/pstip-retrieve-a-redirected-url/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When you look at online documentation for resources and downloadable content, you will often find URLs that redirect you to another place. Most of the web masters prefer this way because the front end redirection URL can be redirected to a right location when there are updates to the content.

There are quite a few examples. To quote a couple of them:

1. SharePoint Server prerequisite software [download links][1]. We cannot use these URLs directly with cmdlets such as _Start-BitsTransfer_. We need to find the right redirected URL for the actual file download.

2. PowerShell 3.0 help content links

<pre class="brush: powershell; title: ; notranslate" title="">Get-Module -Name Microsoft.PowerShell.Management | select -exp HelpInfoUri
</pre>

Now, taking the second example as a use case, let us see how we can retrieve the redirected URL. We can use the System.Net.WebRequest .NET class to achieve this.


    Function Get-RedirectedUrl {
        Param (
            [Parameter(Mandatory=$true)]
            [String]$URL
        )
    
        $request = [System.Net.WebRequest]::Create($url)
        $request.AllowAutoRedirect=$false
        $response=$request.GetResponse()
    
        If ($response.StatusCode -eq "Found")
        {
            $response.GetResponseHeader("Location")
        }
    }
In the above function, we are setting [$request.AllowAutoRedirect][2] to _$false_ to ensure that we don&#8217;t necessarily reach the redirected URL. Our goal is to find the redirection and not really access the redirected URL. Once we set this property, if the given URL redirects to a new location, the &#8220;location&#8221; header contains the redirected URL.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Get-RedirectedUrl -URL 'http://go.microsoft.com/fwlink/?LinkID=210601'
http://download.microsoft.com/download/3/4/C/34C6B4B6-63FC-46BE-9073-FC75EAD5A136/
</pre>

[1]: http://technet.microsoft.com/en-us/library/cc262485.aspx#section4
[2]: http://msdn.microsoft.com/en-in/library/system.net.httpwebrequest.allowautoredirect(v=vs.80).aspx