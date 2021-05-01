---
title: '#PSTip How to combine trace log entries from all SharePoint farm servers using PowerShell'
author: Shay Levy
type: post
date: 2013-11-05T19:00:47+00:00
url: /2013/11/05/pstip-how-to-combine-trace-log-entries-from-all-sharepoint-farm-servers-using-powershell/
categories:
  - SharePoint
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SharePoint

---
**Note**: This tip requires SharePoint 2010 or above.

Whenever there&#8217;s a problem in SharePoint 2010 you are most likely using the [ULS Viewer][1] application to collect trace logs for further details about the problem in question or to monitor machines and the events they create in real-time. ULS Viewer shows information from the server it was launched from. In the case where your farm is behind a load balancer, you can&#8217;t be sure on which server the error occurs and you need to manually process the logs from each server. This can be a very daunting task, especially if your farm contains large number of servers!

Luckily we don&#8217;t need to work that hard&#8211;there&#8217;s a SharePoint cmdlet we can use to gather log information from all farm servers into a single log file which then can be parsed by the ULS Viewer, the _Merge-SPLogFile_ cmdlet. To optimize the performance of this cmdlet it is advised to filter and narrow the results as much as you can by using the parameters of the command (e.g StartTime, EndTime, EventID, etc).

In the following example I&#8217;m using _Merge-SPLogFile_ to combine trace log entries created in the last 15 minutes, from all farm servers, into a single log file on the local computer:

<pre class="brush: powershell; title: ; notranslate" title="">$StartTime = (Get-Date).AddMinutes(-15)
Merge-SPLogFile -Path D:\Logs\FarmMergedLog.log -Overwrite -StartTime $StartTime
</pre>

![](/images/ULS.jpg)

See the Server column to tell from which server an event came from. If the Server column is empty (some viewer application are not capable of displaying it), you can find the server name in the Process column.

[1]: http://archive.msdn.microsoft.com/ULSViewer