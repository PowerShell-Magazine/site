---
title: An example when using Write-Host makes sense
author: Jason Yoder
type: post
date: 2013-06-20T16:00:01+00:00
url: /2013/06/20/an-example-when-using-write-host-makes-sense/
categories:
  - How To
tags:
  - How To

---
In the PowerShell community, there is some debate about the usage of the _**Write-Host**_ cmdlet.  The debate stems from the fact that the output of the _Write-Host_ cmdlet places strings of text on the monitor and not in the pipeline.  Since PowerShell cmdlets are supposed to create objects  some may think you are developing poor code should you be caught using _Write-Host_.  I personally beg to differ.  Although nicely formatted columns can present data in a very orderly package, too much displayed information can hide what is requiring the user’s attention. This is where the Write-Host cmdlet can excel.  For the novice user, the color formatting capabilities of the Write-Host cmdlet can be used to highlight what needs their attention and focuses them on the task at hand.

Recently I presented a project to the Cincinnati PowerShell User Group to provide help desk technicians with many of the answers to the questions that they would not want to ask the end user.  Let’s face it&#8211;either the user does not know the answer, does not want to tell you the truth, or it might be too time-consuming to manually extrapolate the data from an RDP session.  The problem here is that many help desk technicians are not PowerShell savvy.  You need to be aware of the consumers of your code and present information in a way that meets their needs.

In the case of the help desk, we can provide cmdlets to help extract information quickly from a remote client.  We can then have PowerShell flag values that would be considered out of an acceptable range.  This approach would allow the PowerShell-challenged staff to focus on what is requiring their attention without filtering cmdlet output.  The code below is an example of how to extract a client’s hard disk free space.  This simple example will demonstrate using color to flag potential problems.


    Function Get-DiskInfo
    {
        Param(
            $ComputerName = $env:COMPUTERNAME,
            [Switch]$PassThru
        )
        
        Function Get-ColorSplat
        {
            # Create color Splats
            $C1 = @{ForegroundColor="Green";BackgroundColor="DarkGreen"}
            $C2 = @{ForegroundColor="Yellow";BackgroundColor="DarkYellow"}
            $C3 = @{ForegroundColor="White";BackgroundColor="DarkRed"}
            $C4 = @{ForegroundColor="Blue";BackgroundColor="Gray"}
    
            # Create color constants in the previous scope.
            New-Variable -Name "Good" -Value $C1 -Scope 1
            New-Variable -Name "Problem" -Value $C2 -Scope 1
            New-Variable -Name "Bad" -Value $C3 -Scope 1
            New-Variable -Name "Header" -Value $C4 -Scope 1
        } # End: Get-ColorSplat
    
        Function Write-ColorOutput
        {
            Param($DiskInfo)
            
            # Display the headers.
            Write-host "DiskInfo | FreeSpaceGB | PercentFreeSpace"
    
            # Display the data.
            ForEach ($D in $DiskInfo)
            {
                $DeviceID = $D.DeviceID.PadRight(6)
                $FSGB = $D.FreeSpaceGB.ToString().PadRight(6).Remove(5)
                $PFS = $D.PercentFS.ToString().PadRight(6).Remove(5)
    
                If ($D.PercentFS -ge 80)
                { Write-Host "$($DeviceID)   | $($FSGB)       | $($PFS)" @Good }
                ElseIf (($D.PercentFS -lt 80) -and ($D.PercentFS -GE 60))
                { Write-Host "$($DeviceID)   | $($FSGB)       | $($PFS)" @Problem }
                Else
                { Write-Host "$($DeviceID)   | $($FSGB)       | $($PFS)" @Bad }
    
            }
        }
    
        # Get the color splats
        Get-ColorSplat
    
        $DiskInfo = Get-WmiObject Win32_LogicalDisk -ComputerName $ComputerName |
         Select-Object -Property DeviceID,
         @{Name="FreeSpaceGB";Expression={$_.Freespace/1GB}},
         @{Name="PercentFS";Expression={($_.FreeSpace/$_.Size)*100}}
    
         If (!$PassThru) {Write-ColorOutput -DiskInfo $DiskInfo}
         Else {Write-Output $DiskInfo}
    }
This cmdlet has a switch parameter that I want to bring to your attention, _–PassThru_.  Since this code is intended for users who are not avid PowerShell users, the default behaviour is to display the color-coded information.  The usage of the _-PassThru_ parameter suppresses the use of Write-Host and allow an object to be passed to the pipeline.  In this case, we have object in, object out.  By doing this we have the capability of continuing to pass our object into the pipeline.  Here are some examples:

![](/images/Write-Host.png)

In this example we see the default output of color-coded information drawing the technician to the drives that need attention.

```
PS> Get-DiskInfo -ComputerName JasonPC2 -PassThru | Format-Table -AutoSize
DeviceID      FreeSpaceGB        PercentFS
--------      -----------        ---------
C:       34.6056213378906 31.7261065852248
D:       202.438598632813 84.9355612944946
E:       79.9115943908691 33.5278754613268
Z:       83.1089553833008 8.95466512825099
```

In this example, the _–PassThru_ parameter is utilized and we are once again working with objects. The best of both worlds is possible.  Remember to always consider the audience of your code and display information to them that is relevant and fits their skill level.