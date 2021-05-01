---
title: Managing Kemp LoadMaster using REST API and PowerShell
author: Ravikanth C
type: post
date: 2014-01-16T17:00:21+00:00
url: /2014/01/16/managing-kemp-loadmaster-using-rest-api-and-powershell/
categories:
  - How To
  - Kemp
tags:
  - How To
  - Kemp

---
If you are an application or a network administrator, you would have certainly heard about or even used [load balancers][1]. Load balancers are shipped either as hardware or virtual appliances based on the size of the deployment behind them. Today, almost every load balancer vendor offers both varieties. Kemp Technologies, our technology partner and a [new entrant in Gartner magic quadrant][2] for Application Delivery Controllers (ADC) offers both [hardware][3] and [virtual][4] appliances.

While Kemp load balancers have a very intuitive and easy to use web-based user interface for managing load balancing configuration, Kemp also offers a [RESTFul API][5] to script the Kemp load balancer management – whether it is a hardware or a virtual appliance. One thing to remember is that the REST [API interfaces need to be enabled][6] in Kemp Web User Interface (WUI). By default, they are disabled.

Now, with PowerShell, it is very easy to work with REST API. Kemp structured its API in a very easy way. The general syntax for any REST API method in KEMP REST interfaces is something like this:

_https://<LoadMaster IP Address>/access/<command>?<parameter1>=value&<parameter2>=value_

Note that the _Invoke-RestMethod_ and the _Invoke-WebRequest_ have issues with accessing REST API over HTTPS and basic authorization. You will find, over many forums, that there are workarounds. For the purpose of this post, I will use a simple .NET HttpWebRequest.

So, for getting the maximum cache size setting on the KEMP LoadMaster, we can simply call `https://<LoadMaster IPAddress>/access/get?param=cachesize`

This API returns XML output which can be easily parsed using PowerShell.

![](/images/01-Kemp-Browser.png)

Now, let us look at some PowerShell code that can be used to access the cache size setting on the LoadMaster appliance.

```
$uri = "https://192.168.1.3/access/get?param=cachesize"
$credential = Get-Credential -UserName bal

[System.Net.ServicePointManager]::Expect100Continue = $true
[System.Net.ServicePointManager]::MaxServicePointIdleTime = 10000
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
$request = [System.Net.HttpWebRequest]::Create($uri)
$request.Credentials = $Credential

$response = $request.GetResponse()

$stream = $response.GetResponseStream()
if($response.contenttype -eq "text/xml") {
    $Encoding = [System.Text.Encoding]::GetEncoding("utf-8")
    $reader = New-Object system.io.StreamReader($stream, $Encoding)
    $result = $reader.ReadToEnd()
    if ($result.response.success) {
        if ([string]::IsNullOrEmpty($result.response.success.data)) {
             Write-Output $result.response.code
        } else {
             Write-output $result.response.success.data
        }
    }
    else {
         Write-Output $result.response
    }
}
```

![](/images/01-Kemp-CacheSize.png)

As you see, we see the cache size setting on my LoadMaster VM. Now, the above code can be put into a simple and re-usable function to access different methods provided in the Kemp REST API.

If you want to explore more on how to use Kemp REST API and what you can do with it, Kemp provides a [trial license of LoadMaster as a VM][7]. This is available for various hypervisors and VMware Workstation too. Go ahead and give it a try.

Here is a teaser: In our upcoming posts, we will show you how easy it is to manage a Kemp LoadMaster using better ways than writing the functions and dealing with the REST API youself. Stay tuned.

[1]: http://en.wikipedia.org/wiki/Load_balancing_(computing)
[2]: http://kemptechnologies.com/in/news/kemp-technologies-debuts-gartner%E2%80%99s-magic-quadrant-application-delivery-controllers
[3]: http://kemptechnologies.com/in/server-load-balancing-appliances/product-matrix.html
[4]: http://kemptechnologies.com/in/loadmaster-family-virtual-server-load-balancers-application-delivery-controllers
[5]: http://kemptechnologies.com/files/downloads/documentation/7.0/Interface_Descriptions/Interface_Description-RESTful_API.pdf
[6]: http://kemptechnologies.com/loadmaster-70-interface-description-restful-api#2.3
[7]: http://kemptechnologies.com/server-load-balancing-appliances/virtual-loadbalancer/vlm-download