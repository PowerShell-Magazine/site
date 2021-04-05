---
title: '#PSTip Using PowerShell runspaces to find unassigned IPv4 addresses'
author: Ravikanth C
type: post
date: 2015-09-07T15:58:56+00:00
url: /2015/09/07/pstip-using-powershell-runspaces-to-find-unassigned-ipv4-addresses/
views:
  - 16868
post_views_count:
  - 3179
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or later.

We often come across infrastructure configuration scenarios where we need to find a free IP address before assigning the same to a server in an automated manner. However, when the given IP range that is provided is large, it may take longer to complete the free IP scanning. There are many tips and articles written around this topic. There are a few articles that suggest using background jobs and throttling of background jobs. Fellow PowerShell MVP Boe Prox has published a great article on [differences between PowerShell jobs and runspaces][1]. This prompted me to write a function using PowerShell runspaces to get the free IPv4 addresses in a given network range.

If you understand PowerShell runspaces, the following function is self-explanatory. If you are new to PowerShell runspaces, I recommend reading a [series of articles on this by Boe][2].




```powershell
Function Get-AvailableIPv4Address {
    param (
        [System.Collections.ArrayList] $IPRange
    )
    $MaxThreads = 30
    $ScriptBlock = {
        Param (
        [String]$IPAddress
        )
        Test-Connection -ComputerName $IPAddress -Count 1 -Quiet
    }

    $RunspacePool = [RunspaceFactory]::CreateRunspacePool(1,$MaxThreads)
    $RunspacePool.Open()
    $Jobs = @()

    $IPRange | % {
       $Job = ::Create().AddScript($ScriptBlock).AddArgument($_)
       $Job.RunspacePool = $RunspacePool
       $Jobs += New-Object PSObject -Property @{
          IPAddress = $_
          Pipe = $Job
          Result = $Job.BeginInvoke()
       }
    }

    While ( $Jobs.Result.IsCompleted -contains $false) {
        Start-Sleep -Seconds 1
    }

    $Results = @()
    ForEach ($Job in $Jobs)
    {
        if (-not $Job.Pipe.EndInvoke($Job.Result)) {
            $job.IPAddress
        }
    }
}
```

Here is an example of how you use this: 

```powershell
$IPRange = New-Object System.Collections.ArrayList
1 .. 254 | % { $IPRange.Add("192.168.10.$_") | Out-Null }
$IPRange = @(Get-AvailableIPv4Address -IPRange $IPRange)
```

[1]: http://learn-powershell.net/2012/05/13/using-background-runspaces-instead-of-psjobs-for-better-performance/
[2]: http://learn-powershell.net/tag/runspace/