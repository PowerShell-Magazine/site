---
title: Preparing Data for the Report
author: Aleksandar Nikolic
type: post
date: 2012-12-27T17:00:06+00:00
url: /2012/12/27/preparing-data-for-the-report/
categories:
  - News
tags:
  - News

---
One of PowerShell’s most common uses is to collect data and then generate reports from that data. There are a number of different formats in which PowerShell can export data for reporting purposes, such as CSV, XML, and HTML. However, typically the data may only be exported in a raw form and the report made to look pretty using other tools. In this article, based on chapter 12 of* [*PowerShell Deep Dives*](http://www.manning.com/hicks)*, author Jonathan Medd shows how a process block is used to generate the queries that will produce report data for each computer included in the report.

### Preparing Data for the Report

To generate data for a report, I am using a mixture of WMI queries and some standard PowerShell cmdlets to give an example of some typical data you might wish to have in an inventory report. The results of each are stored in variables for later reference and shown in listing 1. This code will be placed in a process block since it will need to execute for every computer we wish to include in the report.

Listing 1 Preparing the inventory queries and variables

```
# Inventory Queries
$OperatingSystem = Get-WmiObject Win32_OperatingSystem -ComputerName $ComputerName
$ComputerSystem = Get-WmiObject Win32_ComputerSystem -ComputerName $ComputerName
$LogicalDisk = Get-WmiObject Win32_LogicalDisk -ComputerName $ComputerName
$NetworkAdapterConfiguration = Get-WmiObject -Query "Select * From Win32_NetworkAdapterConfiguration Where IPEnabled = 1" -ComputerName $ComputerName
$Services = Get-Service -ComputerName $ComputerName
$Hotfixes = Get-HotFix -ComputerName $ComputerName
 
# Variable Build
$Hostname = $ComputerSystem.Name
$DNSName = $OperatingSystem.CSName +"." + $NetworkAdapterConfiguration.DNSDomain
$OSName = $OperatingSystem.Caption
$Manufacturer = $ComputerSystem.Manufacturer
$Model = $ComputerSystem.Model
 
$Resources = [pscustomobject] @{
    NoOfCPUs = $ComputerSystem.NumberOfProcessors
    RAMGB = $ComputerSystem.TotalPhysicalMemory /1GB -as [int]
    NoOfDisks = ($LogicalDisk | Where-Object {$_.DriveType -eq 3} | Measure-Object).Count
    }
...
```

Now we can construct the HTML for each data section using a –Fragment parameter technique. For instance, in the following example I create HTML code for System services info. Note the boldfaced syntax used to sort multiple properties in different directions.

```
$ServicesHTML = $Services | Sort-Object @{Expression="Status";Descending=$true},@{Expression="Name";Descending=

[CA]$false} | Select-Object Name,Status | ConvertTo-Html –Fragment
```

This process will be repeated for each section of the report and will generate HTML code for us to insert into the report later (cut for brevity).

```html
<table>
<colgroup><col/><col/></colgroup>
<tr><th>Name</th><th>Status</th></tr>
<tr><td>Appinfo</td><td>Running</td></tr>
<tr><td>AudioEndpointBuilder</td><td>Running</td></tr>
<tr><td>Audiosrv</td><td>Running</td></tr>
<tr><td>BFE</td><td>Running</td></tr>
<tr><td>BITS</td><td>Running</td></tr>
<tr><td>BrokerInfrastructure</td><td>Running</td></tr>
.................
</table>
...
```

Again though, this is quite static looking data, so why don’t we brighten it up somewhat.

In the example of service data turned into HTML, we will typically end up with services that have a Status of either Running or Stopped. It would be great to differentiate these with color and make them stand out better in our report. Figure 1 illustrates an example of this from the report.

![](/images/ReportData.png)

I achieve this by using the –Replace operator. Any text with <td>Running</td> will be replaced with HTML code to turn the word Running green.

```
$ServicesFormattedHTML =  $ServicesHTML | ForEach {
   $_ -replace "<td>Running</td>","<td style='color: green'>Running</td>"
}
```

I need to assign more than one color though, green for Running and red for Stopped. I can do this without much extra effort however, since I can use multiple –Replace operators on the same line.

```
$ServicesFormattedHTML =  $ServicesHTML | ForEach {
   $_ -replace "<td>Running</td>","<td style='color: green'>Running</td>" -replace "<td>Stopped</td>","<td style=’color: red’>Stopped</td>"
}
```

### Summary

I hope from these PowerShell tips and tricks you have been able to see a few of the possibilities for how you can transform basic HTML reports with black text on white backgrounds using only ConvertTo-HTML, to something more colorful and presentable as a management type report.