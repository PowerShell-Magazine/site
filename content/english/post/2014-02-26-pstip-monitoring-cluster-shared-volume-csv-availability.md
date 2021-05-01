---
title: '#PSTip Monitoring Cluster Shared Volume (CSV) availability'
author: Ravikanth C
type: post
date: 2014-02-26T19:00:34+00:00
url: /2014/02/26/pstip-monitoring-cluster-shared-volume-csv-availability/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
<span style="line-height: 1.5em;">Recently, I was troubleshooting an issue related to CSVs going offline randomly. In that process, I had to detect when a CSV goes offline and then analyse a few log files to find the root cause.</span>

For monitoring CSV availability, I used the WMI events. Here is what I did.

<pre class="brush: powershell; title: ; notranslate" title="">$query = "Select * from __instanceModificationEvent WITHIN 1 WHERE TargetInstance ISA 'MSCluster_Resource' AND TargetInstance.Type='Physical Disk' AND TargetInstance.IsClusterSharedVolume='True' AND TargetInstance.State=3"
Register-WmiEvent -Namespace root\mscluster -Query $query -Action {
    Write-Host "CSV volume $($event.SourceEventArgs.NewEvent.TargetInstance.Name) went offline"
}
</pre>

In the WMI query, I used an intrinsic event registration for the MSCluster_Resource Failover Cluster WMI class. So, whenever an instance of this class gets modified, we get notified. If you want to try bringing the CSV online, you can insert the following line of code into the _-Action_ script block.

<pre class="brush: powershell; title: ; notranslate" title="">Start-ClusterResource -Name $event.SourceEventArgs.NewEvent.TargetInstance.Name
</pre>