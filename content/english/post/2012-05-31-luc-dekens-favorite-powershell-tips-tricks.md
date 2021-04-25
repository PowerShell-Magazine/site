---
title: 'Luc Dekens’ Favorite PowerShell Tips & Tricks'
author: Luc Dekens
type: post
date: 2012-05-31T18:00:28+00:00
url: /2012/05/31/luc-dekens-favorite-powershell-tips-tricks/
views:
  - 15100
post_views_count:
  - 2285
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I was asked to write about 3 PowerShell features I particularly like. Piece of cake I thought. But with the abundance of possibilities available in PowerShell, the selection of my 3 favorite features proved to be more difficult than I imagined. After some reflection, and looking at some recent functions I wrote, I came up with these gems.

### Grouping stuff

I use Group-Object cmdlet quite often when I’m working with vSphere statistics. In [PowerCLI][1] the [Get-Stat][2] cmdlet allows the retrieval of statistical data from a vCenter database. The cmdlet is quite simple to use; pass the name of an entity (virtual machine, hypervisor…) and specify the [metric][3] and a time range. The cmdlet will then return objects that look something like this. The **EntityId** property is a unique identifier for each entity.

![](/images/LucTricks1.png)

When you have to produce a performance report for many thousands of objects, making individual calls for each of these objects to the [Get-Stat][2] cmdlet will be very time-consuming. In other words, your script will take a long time to complete.

Luckily, you can pass more than 1 entity to a [Get-Stat][2] call. The only problem afterwards is how to get the statistical data for each individual entity. And that is where the [Group-Object][4] cmdlet is a life-saver.

You can specify one or more criteria to group your collection of objects. In this case we will use the **EntityId** as a grouping criterion. The returned objects, of type [GroupInfo][5], collect all the matching objects. Under the **Group** property you can access all these matching objects.

![](/images/LucTricks1.png)

From here it is quite easy to produce the report we set out to create.

```
$metrics = "cpu.usage.average","mem.usage.average"
$vms = Get-VM
$start = (Get-Date).AddDays(-1)

Get-Stat -Entity $vms -Stat $metrics -Start $start |
Group-Object -Property EntityId | Foreach-Object{
  New-Object PSObject -Property @{
    VM = $_.Group[0].Entity.Name
    Date = $_.Group[0].Timestamp.Date
    CPU = $_.Group | Where-Object {$_.MetricId -eq $metrics[0]} |
          Measure-Object -Average -Property Value |
          Select -ExpandProperty Average
    Memory = $_.Group | where {$_.MetricId -eq $metrics[1]} |
          Measure-Object -Average -Property Value |
          Select -ExpandProperty Average
  }
} | Select VM,Date,CPU,Memory |
Export-Csv .\report.csv -NoTypeInformation -UseCulture
```

Note that we use the **Group** property combined with a [Where-clause][6] to retrieve all the objects, repectively for the **CPU** and the **Memory** metric. If you want to read a bit more on vSphere statistics with PowerCLI have a look at my [statistics blog posts][7].

### Living in another time

This tip is something that came from one of my [recent posts][8]. I wanted to query the search engine on a [Jive Forum][9]. In the function I wanted to be able to specify a **Start** and **Finish** timestamp for the search. The problem was that the [Jive Forum][9] software wants these timestamp in it’s **local timezone,** which was in this peticular case **PST**. Annoying when you run the script from another timezone.

The solution in this case was to use one of the .Net methods that is available in your PowerShell session. The [TimeZoneInfo][10] class contains a method, called [FindSystemTimeZoneById][11], that allows you to retrieve the characteristics of a timezone by name.

By taking the local time and then converting it to the time of the timezone you are interested in, it becomes just a matter of retrieving the time difference between these two to know the actual time difference between the timezone you are in and the other timezone.

<pre>$pst = [TimeZoneInfo]::FindSystemTimeZoneById("Pacific Standard Time")
$t0 = Get-Date
$t1 = [TimeZoneInfo]::ConvertTime($t0,$pst)
$tdiff = $t0 - $t1
$tdiff.Hours</pre>

Nothing spectacular, just handy when working in different timezones. And it shows how important it is to know about all the **.Net classes** you can easily access and use from PowerShell.

### Pick one!

Sometimes you receive a collection of objects and you need to pick one. But which one? You don’t always want to use the first object or the last object. That’s where the [Get-Random][12] cmdlet can help. Besides picking a random number, you can also use the [Get-Random][12] cmdlet to pick an object out of stream of objects:

<pre>Get-Service | Get-Random</pre>

Each time you run this, it is very likely that another service will be returned. But can we be sure that this will spread the selection of an object evenly over the available objects? Let’s use PowerShell to check.

<pre>$count = @{}
for($i = 0;$i -lt 1000;$i++){
	$rnd = Get-Random -Minimum 1 -Maximum 10
	$count[$rnd] += 1
}
$count.GetEnumerator() |
Sort-Object -Property Name |
Out-Chart -Label Name -Values Value -Title "Random distribution"</pre>

This code randomly picks a number between 1 and 10, and does this a thousand times. Each time a specific number is selected, the script will increment the corresponding value in the hash table. And since a picture says more than a thousand words, I used one of the [PowerGadgets][13] cmdlets to visualize the result.

![](/images/LucTricks3.png)

The graph clearly shows that each of the numbers 1 to 10 was selected within a reasonable margin around the ideal number of a 100 selections. Good enough for me (and my scripts), I’ll pick my objects randomly from now on!

[1]: http://communities.vmware.com/community/vmtn/server/vsphere/automationtools/powercli?view=overview
[2]: http://www.vmware.com/support/developer/PowerCLI/PowerCLI501/html/Get-Stat.html
[3]: http://pubs.vmware.com/vsphere-50/topic/com.vmware.wssdk.apiref.doc_50/vim.PerformanceManager.html
[4]: http://technet.microsoft.com/en-us/library/dd347561.aspx
[5]: http://msdn.microsoft.com/en-us/library/ms584694%28v=vs.85%29
[6]: http://technet.microsoft.com/en-us/library/dd347549.aspx
[7]: http://www.lucd.info/?s=powercli+statistics+part
[8]: http://www.lucd.info/2012/02/29/automate-your-vmtn-search/
[9]: communities.vmware.com
[10]: http://msdn.microsoft.com/en-us/library/system.timezoneinfo.aspx
[11]: http://msdn.microsoft.com/en-us/library/system.timezoneinfo.findsystemtimezonebyid
[12]: http://technet.microsoft.com/en-us/library/ff730929.aspx
[13]: http://www.softwarefx.com/powergadgets/