---
title: Handle log files with PowerShell 3.0
author: Niklas Akerlund
type: post
date: 2012-12-06T19:00:18+00:00
url: /2012/12/06/handle-log-files-with-powershell-3-0/
categories:
  - How To
tags:
  - How To

---
In a recent task I had to handle quite a massive amount of log files from a load test program and I wanted to get the amount of log records and also the time from the first to the last record. With this data I could calculate the average per second of the jobs that was processed. Doing this manually would have taken forever. For each log file there were about 20 records. Each process created its own log file and we had 625 processes on each server.

The test aimed on looking at the difference between running this program on one virtual server with 1,2,3,4,5,6,7, and 8 CPUs and then two virtual servers with 8 CPUs and then four servers with 4 CPUs and last four servers with 8 CPUs. The program was created to load the CPU with calculations and see how the OS would handle that load, and also the goal was to test the single physical machine that was hosting these virtual machines. The physical server hardware had 2 CPUs with 8 cores and HT enabled; that was why we aimed at maxing out all 16 cores and 32 logical CPUs.

The log files was updated with a record after 1 x 10<sup>9</sup> computations and I then counted number of records and when each process had started and also the time of the last record of each process. This was used to get the average number of computations per second overall processes.

The folders containing the log files with a single server did not have subfolders and the ones with up to four servers had subfolders, that is why I am using the _–File_ and _–Recurse_ on my _Get-ChildItem_ cmdlet to get the log files.

![](/images/image1.jpg)

As you can see I had some test folders to traverse and also some log files.

![](/images/image2.jpg)

And here is an extract from one of the log files:

![](/images/image3.jpg)

Here is my reporting script:


    #####################################
    # Handle log files and get average iterations per second
    #
    # Niklas Åkerlund 2012-10-30
    #####################################
    $folders = Get-ChildItem C:\testlog\test -Directory
    $TotalReport = @()
    foreach ($folder in $folders){
        $filelist = (Get-ChildItem $folder.FullName -Recurse -File).FullName
    $allvalues = @()
    $startReport = @()
    
    foreach ($file in $filelist){
    
        $filecontent = Get-Content $file
        $Count = $filecontent.Count
        for ($i=4;$i -le ($Count -1);$i++){
            $line = $filecontent[$i]
            $array = $line.Split(" ")
            $seconds = [double]$array[4].Replace(",",".")
            $iterSeconds = [double]$array[10].Replace(",",".")
            $logObj = [pscustomobject]@{
                Seconds= $seconds
                Iterations=$array[6]
                IterSeconds=$iterSeconds
            }
            $allvalues +=$logObj
    
        }
        # Get all logtimes
        $reportObj = [pscustomobject]@{
            Test=($folder.Name).Split("_")[2]
            Process = [int](($file.Split("\")[-1]).Split("_")[2]).Split(".")[0]
            Start= ($filecontent[3]).Split(" ")[0] + " " + ($filecontent[3]).Split(" ")[1]
            LastRun=($filecontent[($Count-1)]).Split(" ")[1] + " " + ($filecontent[($Count-1)]).Split(" ")[2]
            TotalRunPerFile= $Count -5
            }
    
        $startReport +=$reportObj
    
    }
    
    # Count the average iterations for all processes on the given time
    $FirstStart = $StartReport | Sort-Object Start | Select-Object -First 1 Start
    $LastFinish = $StartReport | Sort-Object LastRun | Select-Object -Last 1 LastRun
    
    $totalRunnings = ($StartReport.TotalRunPerFile | Measure-Object -Sum).Sum
    
    $totalsec = ((get-date $LastFinish.LastRun) - (get-date $FirstStart.Start)).TotalSeconds
    
    $avg = ($totalRunnings*1000000000)/$totalsec
    
    $TotSec = $allvalues.Seconds | Measure-Object -Sum
    
    $AllDataObj = [pscustomobject]@{
        Name = ($folder.Name).Split("_")[2]
        TotSecProgram=$TotSec.Sum
        TotalSec=$totalsec
        #MinSec=$avgSec.Minimum
        AvgIterTotSec=$avg
        first=$FirstStart.Start
        last=$LastFinish.LastRun
        duration=((get-date $LastFinish.LastRun) - (get-date $FirstStart.Start)).TotalHours
        Iterations=$totalRunnings
        AllValues=$allvalues
    }
    $TotalReport += $AllDataObj
    }
    $TotalReport
I used the data from this to export it to a CSV file and then open it in Excel to create a graph from the _Name_ and the _AvgIterTotSec_.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $logdata = .\logmassage2.ps1
PS&gt; $logdata | Select-Object Name,AvgIterTotSec,duration,NumRuns | Export-Csv -Path C:\testlog\logdata.csv -UseCulture -NoTypeInformation
</pre>

The conclusion on this test is that with this load testing software aimed at maxing out the CPUs we did not get any performance benefit of using more logical CPUs than there was physical cores on the hardware, rather a decrease of performance as the graph is showing.

![](/images/image004.jpg)

The first 8 pillars shows the values from a single virtual server and the 4 last pillars show the results for several virtual servers. What I am trying to show with the glowing pillar is, that test has the highest result and the configuration was two virtual servers with 8 CPUs each, the test after that with three virtual servers with 8 CPUs each and one virtual server with 7 CPUs and this shows that with over allocating the physical cores gives a result that is performing worse. The reason for leaving one CPU for the virtual host was because of old best practices guidelines and we wanted to show that this has no effect on the result and so to say bust that myth.

As you can see in my script I am using the _Measure-Object_ and also the PowerShell 3.0 ability to get a value from an array directly without using a foreach or for loop. Maybe not the most perfect script written but it got the job done and that is what I love with PowerShell and its ability to easily get the results wanted and that without too much pain .