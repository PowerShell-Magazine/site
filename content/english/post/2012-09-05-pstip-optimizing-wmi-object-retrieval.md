---
title: '#PSTip Optimizing WMI object retrieval'
author: Ravikanth C
type: post
date: 2012-09-05T18:00:15+00:00
url: /2012/09/05/pstip-optimizing-wmi-object-retrieval/
views:
  - 8002
post_views_count:
  - 1792
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Whenever we use WMI cmdlets to query either local or remote machines, we need to keep the following points in mind:

Don’t ever pipe the WMI objects to Where-Object cmdlet to filter the WMI output unless there is a need to do so. Consider this example:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Class Win32_OperatingSystem -ComputerName $servername | Where-Object { $_.BuildNumber -eq "7601" }
</pre>

The above command forces WMI to retrieve the objects to the local session and pass them on to Where-Object for filtering. This takes longer depending on the number of objects to retrieve or the size of the objects. This is where the –Filter parameter of the Get-WmiObject comes into play.

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Class Win32_OperatingSystem -ComputerName $servername -Filter "BuildNumber=7601"
</pre>

In this example using –Filter parameter, the filtering happens on the remote system and results in faster execution of the command.

![](/images/tip3-measure.png)