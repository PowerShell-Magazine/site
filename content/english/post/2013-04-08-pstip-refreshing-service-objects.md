---
title: '#PSTip Refreshing service objects'
author: Shay Levy
type: post
date: 2013-04-08T18:00:24+00:00
url: /2013/04/08/pstip-refreshing-service-objects/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When you assign the result of a service query to a variable, you must take into account one very important thing &#8211; the result is just a snapshot of the service state for a specific point in time.

```
PS> $svc = Get-Service -Name W3SVC
PS> $svc

Status   Name               DisplayName
------   ----               -----------
Running  W3SVC              World Wide Web Publishing Service
```

Here you can see that the service Status is _&#8216;Running&#8217;_. However, if the state has been changed outside your script (you can simulate it by stopping the service via the _services.msc_ MMC snap-in), the state of your variable will still show _&#8216;Running&#8217;_ and your script may fail or perform steps that are not to be run in the current state.

To have the most fresh settings of the service before making such descions you can re-query the service, but there&#8217;s a better way, use its [_Refresh_ method][1].

<pre class="brush: powershell; title: ; notranslate" title="">$svc.Refresh()
</pre>

Calling the _Refresh_ method refreshes the service property values to their current values.

[1]: http://msdn.microsoft.com/en-us/library/system.serviceprocess.servicecontroller.refresh.aspx