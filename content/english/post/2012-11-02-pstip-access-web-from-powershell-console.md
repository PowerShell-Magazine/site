---
title: '#PSTip Access web from PowerShell console'
author: David Moravec
type: post
date: 2012-11-02T18:00:09+00:00
url: /2012/11/02/pstip-access-web-from-powershell-console/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
As I am still running PowerShell 2.0 on my work computer, I can’t use the **Invoke-WebRequest** cmdlet. This cmdlet is introduced in Windows PowerShell 3.0. When I need to download some web page, I still use my old .NET friend&#8211;**Net.WebClient** class.

The following command will download the Bing home page:

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object Net.WebClient).DownloadString(‘http://www.bing.com’
</pre>

As I use this class frequently, I’ve created a function for that and I am exposing a Net.WebClient object as a variable to my global scope.


    function New-WebClient
    {
        $wc = New-Object Net.WebClient
        $wc.UseDefaultCredentials = $true
        $wc.Proxy.Credentials = $wc.Credentials
    
        $wc.Encoding = [System.Text.Encoding]::UTF8
        $wc.CachePolicy = New-Object System.Net.Cache.HttpRequestCachePolicy([System.Net.Cache.HttpRequestCacheLevel]::NoCacheNoStore)
    
        Write-Output $wc
    }
    $Global:wc = New-WebClient
As you can see I’ve set also some other properties to satisfy my needs:

  1. Use of default credentials. As we user proxy with authentication, I need to pass my actual credentials to proxy.
  2. Setting encoding to UTF8
  3. I am not using cache.

At the end of New-WebClient function, I send the object out and assign it to a global $wc variable. I can use it later as in the following command:

<pre class="brush: powershell; title: ; notranslate" title="">$web = $wc.DownloadString('http://www.bing.com')
</pre>

I know that some people uses this technique to process RSS channels. I don’t use it for this scenario, but you can download RSS channel, convert it to XML and then process in console. Frequent usage is:

```
PS> $rss =  $wc.DownloadString('http://feeds.feedburner.com/PowershellMagazine')
PS> $rss.rss.channel.item | Format-Table Title

title
-----
Two new PowerShell modules related to Storage Spaces
#PSTip How to speed up the Test-Connection command
#PSTip Get system power information
#PSTip Wait for executable to finish
#PSTip Converting numbers to binary and back
Using SkyDrive to sync your WindowsPowerShell folder
#PSTip Get your reboot history
#PSTip Converting numbers to HEX
Connecting to Hyper-V virtual machines with PowerShell
Manipulating Wildcards
```