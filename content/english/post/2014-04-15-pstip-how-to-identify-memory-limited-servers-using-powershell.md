---
title: '#PSTip How to identify “memory limited” servers using PowerShell'
author: Shay Levy
type: post
date: 2014-04-15T18:00:01+00:00
url: /2014/04/15/pstip-how-to-identify-memory-limited-servers-using-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 2.0 or above.

Yesterday, I got a call from one of our BI managers. Listen, he says, I suspect that one of my BI servers is working too hard. It&#8217;s a physical server with 64GB of RAM and I think it doesn&#8217;t utilize all of its memory.

We were able to determine that the installed OS was actually a wrong one. We had Server 2008 R2 installed, and according to [this article][1], Standard editions (of 2008 R2) are limited to 32GB. I took a quick look the values in Task Manager and I could verify that the total memory was 32GB.

![](/images/memory1.png)

A second look at the system properties page confirmed the above.

![](/images/memory2.png)

With that, I continued to the next step of upgrading the servers to Enterprise edition. Luckily I did not have to re-install the server. I followed [this article][2]  and used DISM for the upgrade. Using the following script I found 3 more servers in my environment that had more physical memory than the OS could see.

    $computers = Get-ADComputer -Filter {OperatingSystem -like "*standard*"} -Properties OperatingSystem | Select-Object Name,OperatingSystem
    
    foreach($computer in $computers)
    {
        if(Test-Connection -ComputerName $computer.Name -Count 1 -Quiet)
        {
            $total = (Get-WmiObject Win32_PhysicalMemory -ComputerName $computer.Name | Measure-Object Capacity -Sum).Sum/1GB
            $visible = (Get-WmiObject Win32_OperatingSystem -ComputerName $computer.Name).TotalVisibleMemorySize/1MB
            if( ($total-$visible) -gt 2)
            {
                New-Object PSObject -Property @{
                    Name = $computer.Name
                    OS = $computer.OperatingSystem
                    Memory = $total
                    Visible = $visible
                }
            }
    	}
    }
The code get&#8217;s all standard servers from AD and checks if the total amount of RAM is larger, by 2GB, from the amount of visible ram. The result of _TotalVisibleMemorySize_ does not give absolute numbers. You might get 31.7 when you have 32GB of RAM. So a difference of 2GB will filter out all those &#8220;almost&#8221; values.

[1]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa366778(v=vs.85).aspx#physical_memory_limits_windows_server_2008_r2
[2]: http://blogs.technet.com/b/server_core/archive/2009/10/14/upgrading-windows-server-2008-r2-without-media.aspx