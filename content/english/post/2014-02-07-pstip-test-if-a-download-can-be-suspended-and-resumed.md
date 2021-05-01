---
title: '#PSTip Test if a download can be suspended and resumed'
author: Ravikanth C
type: post
date: 2014-02-07T19:00:01+00:00
url: /2014/02/07/pstip-test-if-a-download-can-be-suspended-and-resumed/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Today, most of the web servers that provide downloads enable resumable downloads. Of course, there are still sites that don&#8217;t offer resumable downloads. When using BITS transfer cmdlets, we can suspend and resume downloads using the _Suspend-BitsTransfer_ and _Resume-BitsTransfer_ cmdlets. However, if the web server does not support resumable downloads, trying to resume a download using BITS transfer cmdlets will have no impact.

This is where we can inspect the HTTP response headers to test if a server supports resuming downloads. The HTTP header that tells us this information is &#8220;[Accept-Ranges][1]&#8220;. The value of this header will be set to none if the server does not supporting resuming downloads. The following function helps us in testing for resumable downloads.

<pre class="brush: powershell; title: ; notranslate" title="">Function Test-ResumableDownload {
    param (
       [String]$url
    )
    $request = [System.Net.WebRequest]::Create($url)
    $request.Method = "GET"
    $result = $request.GetResponse()
    $result.Headers["Accept-Ranges"] -ne "none"
}
</pre>

We can use this technique as a prerequisite before starting a download using the _Start-BitsTransfer_ cmdlet to let the user know whether a download can be resumed or not.

[1]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html