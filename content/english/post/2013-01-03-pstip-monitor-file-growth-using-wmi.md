---
title: '#PSTip Monitor file growth using WMI'
author: Ravikanth C
type: post
date: 2013-01-03T19:00:38+00:00
url: /2013/01/03/pstip-monitor-file-growth-using-wmi/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 2.0 or above.

When running long running scripts, we prefer to log the activities performed to a log file. By the very nature of these tasks, you may find yourself in a situation where you need to truncate the log file and start writing to a new log location or completely stop logging when we find that the log is becoming larger than it should be. This is usually done to preserve the readability of the log files and also, in some cases, to ensure we don&#8217;t use up all available disk space.

There is no built-in cmdlet to do this. But, thankfully, WMI provides events we can use to achieve this. Let us see how:

```
$query = "Select * from __InstanceModificationEvent WITHIN 5 WHERE TargetInstance ISA 'CIM_DataFile' AND TargetInstance.Name='C:\\Logs\\test.log'"

Register-WmiEvent -Query $query -Action {
    Write-Host "Current file size is: " $Event.SourceEventArgs.NewEvent.TargetInstance.FileSize
    $prevSize = $Event.SourceEventArgs.NewEvent.PreviousInstance.FileSize
    $curSize = $Event.SourceEventArgs.NewEvent.TargetInstance.FileSize
    if ($curSize -gt $prevSize) {
        $bytes = $curSize - $prevSize
        Write-Host "File grew by: $bytes bytes"
    } else {
        $bytes = $prevSize - $curSize
        Write-Host "File reduced by: $bytes bytes"
    }
}
```

In the above snippet, we used _[CIM_DataFile][1]_ WMI class to monitor the changes made to a log file at C:\Logs\test.log. The WMI events contain two instances of the WMI class we are looking at. This includes a _PreviousInstance_ and a _TargetInstance_ &#8212; which represents the most up-to-date instance. The _CIM_DataFile_ class provides a property called _FileSize_ which can be used to determine (using the Previous and Target instances) whether the file grew in size or reduced.

When this registered WMI event triggers, we can see the changes to the file size at the console.

![](/images/event.png)

Although we used a simple Write-Host method to show the file size changes, in a proper implementation we may want to use the decision block to perform something like truncating the existing log file, etc.

Make a note that the event subscriptions made using _Register-WMIEvent_ cmdlet are temporary subscriptions. This means that whatever action we have defined in the script block won&#8217;t be executed once the PowerShell host is closed. However, you can use WMI [Permanent event consumers][2] to use WMI events irrespective of the PowerShell host state.

[1]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa387236(v=vs.85).aspx
[2]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa393013(v=vs.85).aspx#event_consumers