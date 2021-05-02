---
title: Parsing remote machine local date and time with WMI
author: Shay Levy
type: post
date: 2012-04-20T18:00:50+00:00
url: /2012/04/20/parsing-remote-machine-local-date-and-time-with-wmi/
views:
  - 20683
post_views_count:
  - 4279
categories:
  - WMI
tags:
  - WMI

---
The other day I was asked to check the local date and time on some workstations in our network. The reason for that was time skew, and as I was told, some workstations’ clock was 1 hour behind. I launched PowerShell and wrote a quick script to get the raw WMI value of the local date and time on a list of ****remote machines and convert it to a readable DateTime object:

```powershell
$computers = 'pc1','pc2','pc3'
$computers | ForEach-Object{
   $os = Get-WmiObject -Class Win32_OperatingSystem -ComputerName $_
   $cdt = @{Name='ConvertedDateTime';Expression={$_.ConvertToDateTime($os.LocalDateTime)}}
   $os | Select-Object -Property Name,LocalDateTime,$cdt
}
```


The result looked like:

```shell
Name  LocalDateTime              ConvertedDateTime
----  -------------              ----
pc1   20110912093710.927000+120  9/12/2011 10:37:10 AM
pc2   20110912103722.322000+120  9/12/2011 10:37:22 AM
pc3   20110912103703.530000+120  9/12/2011 10:37:03 AM
```


I took a quick look at the last column and was able to determine that the time on the machines in question was correct. Right?

Wrong! Take a look at the result. Do you find anything unusual? Sharp-eyed among you probably noticed that there’s a one hour difference on pc1, between the hour in the LocalDateTime and the converted one (9 vs. 10). I was always under the impression that the DMTF conversion happens on the remote machine but apparently I was wrong.

The ConvertToDateTime method used in the calculated property is a script method added via ETS (Extended Type System) and under the cover it is using the [System.Management.ManagementDateTimeconverter]::ToDateTime method.

The description of the method in MSDN says:

Converts a given DMTF date time to DateTime. The returned DateTime will be in the current time zone of the system. 

Which raises the question: the current time zone of the remote machine or the time zone of the machine that executed the script? The ToDateTime method doesn&#8217;t support remote connections so conversion is done locally. This can, and does, introduce a &#8220;bug&#8221; or false results. No one really checks the value of LocalDateTime (guilty as charged); it&#8217;s too cryptic, and I have always counted on the result of the ConvertToDateTime method.

Now that I know that the conversion is made on the machine that executes the script I use one of the following to parse the LocalDateTime value (the Win32_LocalTime class is supported on XP/Windows Server 2003 and above):

```powershell
PS> $lt = Get-WmiObject -Class Win32_LocalTime -ComputerName pc1
PS> Get-Date -Year $lt.Year -Month $lt.Month -Day $lt.Day -Hour $lt.Hour -Minute `
              $lt.Minute -Second $lt.Second
Monday, September 12, 2011 9:37:10 AM
or
PS> [datetime]::ParseExact($lt.split('.')[0],'yyyyMMddHHmmss',$null)
Monday, September 12, 2011 9:37:10 AM
```