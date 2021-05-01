---
title: '#PSTip Offer Delayed Restart with Abort Functionality'
author: Manoj Ravikumar
type: post
date: 2013-09-27T18:00:33+00:00
url: /2013/09/27/pstip-offer-delayed-restart-with-abort-functionality/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Restart-Computer, no surprise, does the job of restarting a computer but there are times when you want to add the functionality to abort a restart. Something along the lines of &#8220;Restarting computer in 30 seconds, Press any key to cancel…&#8221;. It reminds me of good, old days of batch scripting.

The function _Restart-ComputerWithDelay_ is a quick and dirty way to achieve this functionality and also provides a countdown timer.


    Function Restart-ComputerWithDelay {
        Param(
            [int]$TimeOutinSeconds = 30,
            [string[]]$ComputerName = $env:COMPUTERNAME
        )
    	Write-Host "Press any key to abort. Restarting in $TimeOutinSeconds" -NoNewLine
    
        While ($TimeOutinSeconds -gt 0 -and -not $host.UI.RawUI.KeyAvailable) {
            Start-Sleep -Seconds 1
            $TimeOutinSeconds --
            Write-Host –NoNewLine ",$TimeOutinSeconds"
        }
    
        if ($Host.UI.RawUI.KeyAvailable -eq $false) {
            Write-Host "`nRestarting Computer(s)..."
            Restart-Computer -ComputerName $ComputerName -Force
        }
        else {
            $Host.UI.RawUI.FlushInputBuffer()
            Write-Host "`nRestart Aborted!"
        }
    }
    
    Restart-ComputerWithDelay
    # Restart-ComputerWithDelay –TimeOutinSeconds 15 –ComputerName "Demo1","Demo2"
With a slightly different logic and with less lines of code, you can implement the above functionality using the Ctrl + C key combination.

<pre class="brush: powershell; title: ; notranslate" title="">Write-Host "Restarting in 30 seconds. Press CTRL-C to abort or any key to restart…"
</pre>