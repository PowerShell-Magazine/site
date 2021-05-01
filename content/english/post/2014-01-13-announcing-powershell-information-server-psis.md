---
title: Announcing PowerShell Information Server (PSIS)
author: Ravikanth C
type: post
date: 2014-01-14T04:01:39+00:00
url: /2014/01/13/announcing-powershell-information-server-psis/
categories:
  - News
tags:
  - News

---
[Johan Akerstrom][1] &#8211; PowerShell expert from Sweden &#8211; released a PowerShell module that can be used to start a PowerShell-based web server. He preferred to call it a PowerShell Information Server (PSIS). Although, I am not really sure about the name given to it, I completely love the concept and work he did.

This module is available on [Github][2]. It is very easy to use and work with.

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-PSIS -URL "http://*:8087/" -AuthenticationSchemes negotiate -ProcessRequest {
    Write-Verbose $request.RequestBody
    Get-Service $request.RequestObject.ServiceName
} -Verbose -Impersonate
</pre>

This gets the web server started. The _-ProcessRequest_ is what gets executed when a request arrives at the web server. So, in my example, I am just getting the service information.

We can call the end point we just started using the _Invoke-RestMethod_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">$data = [pscustomobject]@{
   ServiceName = "winmgmt"
}
$postData = $data | ConvertTo-Json
Invoke-RestMethod -Method post -Uri 'http://localhost:8087/json' -UseDefaultCredentials -Body $postData | ConvertTo-Json
</pre>

I already have a list of things I want to try. Go ahead and try the module and leave your feedback for Johan.

[1]: http://blog.cosmoskey.com/about/
[2]: https://github.com/CosmosKey/PSIS