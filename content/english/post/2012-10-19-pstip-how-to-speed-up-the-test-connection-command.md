---
title: '#PSTip How to speed up the Test-Connection command'
author: Peter Kriegel
type: post
date: 2012-10-19T18:00:35+00:00
url: /2012/10/19/pstip-how-to-speed-up-the-test-connection-command/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note:** This tip requires PowerShell 2.0 and above

If you only like to know if a computer can be contacted across a network, you can use the Test-Connection cmdlet with the –Quiet parameter. You can even send only one echo request packet (ping) to the computer by using -Count 1.

This is much faster than executing Test-Connection without any parameters (which sends 4 echo requests by default), but if you have an unstable network connection or the computer is very busy, the ping reply can get lost!

To solve this situation, you can send more than one test which cost more time to wait for. To get the fastest reply possible, you can use a loop with a break or a function with a return statement, to get the earliest possible response. The delay time between every echo request in the Test-Connection cmdlet is set to the time interval in seconds. If you like to lower this interval you can use the Start-Sleep command.


    Function Test-ConnectionQuietFast {
        [CmdletBinding()]
        param(
            [String]$ComputerName,
            [int]$Count = 1,
            [int]$Delay = 500
        )
    
        for($I = 1; $I -lt $Count + 1 ; $i++)
        {
            Write-Verbose "Ping Computer: $ComputerName, $I try with delay $Delay milliseconds"
    
            # Test the connection quiet to the computer with one ping
            If (Test-Connection -ComputerName $ComputerName -Quiet -Count 1)
            {
                # Computer can be contacted, return $True
                Write-Verbose "Computer: $ComputerName is alive! With $I ping tries and a delay of $Delay milliseconds"
                return $True
            }
    
            # delay between each pings
            Start-Sleep -Milliseconds $Delay
        }
    
        # Computer cannot be contacted, return $False
        Write-Verbose "Computer: $ComputerName cannot be contacted! With $Count ping tries and a delay of $Delay milliseconds"
        $False
    }
    
    # Example:
    Test-ConnectionQuietFast -ComputerName localhost -Count 3 -Delay 300 –Verbose