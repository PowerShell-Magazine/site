---
title: '#PSTip Retrieve a redirected URL – PowerShell 3.0 way!'
author: Ravikanth C
type: post
date: 2013-01-31T19:01:43+00:00
url: /2013/01/31/pstip-retrieve-a-redirected-url-powershell-3-0-way/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note:** This tip requires PowerShell 3.0 or above.

In an [earlier tip][1], we looked at how we can retrieve the redirected URL using a .NET class. In today&#8217;s tip, we will look at how we can simplify that process using an in-built PowerShell 3.0 cmdlet &#8211; _Invoke-WebRequest_.

```
$uri = 'http://go.microsoft.com/fwlink/?LinkID=210601'
$request = Invoke-WebRequest -Uri $uri -MaximumRedirection 0 -ErrorAction Ignore

if($request.StatusDescription -eq 'found')
{
   $request.Headers.Location
}
```

This is it. Much simpler than the earlier approach!

[1]: /2013/01/29/pstip-retrieve-a-redirected-url/