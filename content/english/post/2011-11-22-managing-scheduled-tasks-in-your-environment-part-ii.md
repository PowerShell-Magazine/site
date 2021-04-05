---
title: Managing scheduled tasks in your environment – Part II
author: Claus Nielsen
type: post
date: 2011-11-22T10:21:20+00:00
url: /2011/11/22/managing-scheduled-tasks-in-your-environment-part-ii/
views:
  - 25313
post_views_count:
  - 2542
categories:
  - How To
tags:
  - How To
  - Scheduled Tasks

---
[In Part 1](/2011/11/21/managing-scheduled-tasks-in-your-environment-part-i/) we looked at gathering information about the scheduled tasks in your environment, in part 2 we will look at how to make changes to the scheduled tasks, and fixing a “problem” with scheduled tasks created in the 2008(R2) Task Scheduler GUI.

### Changing passwords on scheduled tasks

Again we turn our attention to PowerShell and schtasks.exe.  schtasks.exe has a parameter called /change, which as the name implies, allows you to change a scheduled task. As I am looking at this from a security perspective, the only thing the script is able to change is the username and password.

```powershell
function Set-ScheduledTask
{ 
    [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='High')]

    param
    (
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [Alias("HOSTNAME")]
        [String[]]
        $ComputerName,
        
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [Alias("Run As User")]
        [String[]]
        $RunAsUser,
               
        [Parameter(Mandatory = $true)]
        [String[]]
        $Password,
        
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [String[]]
        $TaskName
    )            

    Process {
        Write-Verbose "Updating: $($_.'TaskName')"
        if ($pscmdlet.ShouldProcess($computername, "Updating Task: $TaskName "))
        {
            Write-Verbose "schtasks.exe /change /s $ComputerName /RU $RunAsUser /RP $Password /TN `"$TaskName`""
            $strcmd = schtasks.exe /change /s "$ComputerName" /RU "$RunAsUser" /RP "$Password" /TN "`"$TaskName`"" 2>&1
            Write-Host $strcmd
        }
    }
}
```

The script accepts pipeline input, so you can pipe your results directly from Get-ScheduledTask to Set-ScheduledTask  (Make sure you have narrowed down the result of Get-ScheduledTask, to only the tasks you want to change, otherwise, you  will end up with a lot of tasks that won’t run).

As an added security against accidentally changing all scheduled task across your domain, I am using another PowerShell v2  advanced function functionality :

```powershell
PS> [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = 'High')]
```

This tells PowerShell that the function supports the –WhatIf and –Confirm parameters, so if you do:

```powershell
PS> Get-ScheduledTask | Set-ScheduledTask –Password VerySecurePWD –WhatIf
```

It will output to the screen which tasks it is going to try to change, but not actually do anything. Also,setting ConfirmImpact=&#8217;High&#8217;, means that you will be prompted for a confirmation before any task is changed.  If you are really sure what you are doing, or want to run the script unattended, you can always supply –Confirm:$False to the function like this:

```powershell
PS> Get-ScheduledTask | Set-ScheduledTask –Password VerySecurePWD –Confirm:$false
```

This will change the password on all the scheduled tasks found by Get-ScheduledTask. You can also use Set-ScheduledTask with parameters, if you just want to change a single scheduled task.

```powershell
PS> Set-ScheduledTask -Computername localhost -TaskName mytask -RunAsUser mydomain\Administrator -Password VerySecurePWD
```


The above example will change the scheduled task mytask on the local machine to run under the Administrator account in mydomain with a password called VerySecurePWD.

One last thing I would like to mention, which I spent quite a lot of time figuring out, is that scheduled tasks created in the GUI on Windows Server 2008 and Server 2008 R2 cannot readily be changed by schtasks.exe. After a lot of digging, I found that when you create a scheduled task in the GUI, it uses a time format which uses 7 decimals for showing seconds (I guess that is down to 1.000.000<sup>th</sup> of a second), but schtasks.exe cannot handle that format. So, when you try to change a scheduled task that was created in the GUI, it will give you a following error message:

ERROR: The parameter is incorrect.

So, if you are seeing error messages like that one, it is probably because tasks were created from the GUI.

The only way I have found to “fix” this issue is to export the task as an XML file, then alter the XML directly and then re-import the scheduled task overwriting the existing one.

```powershell
Function Set-2k8ScheduledTask
{
    [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='High')]            
    param
    (
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [Alias("HOSTNAME")]
        [String[]]
        $ComputerName,
        
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [String[]]
        $TaskName
    )            

    Begin
    {
        $ShortDate = [regex]"(?[0-9]{4}[/.-](?:1[0-2]|0[1-9])[/.-](?:3[01]|[12][0-9]|0[1-9])T(?:2[0-3]|[01][0-9])[:.][0-5][0-9][:.][0-5][0-9])(?\.\d*)"
    }           

    Process {
        $XMLIn = schtasks /query /s $ComputerName /tn $TaskName /xml
        if (Test-Path "$Env:TEMP\$($TaskName).xml")
        {
            Remove-Item "$Env:TEMP\$($TaskName).xml"
        }            

        foreach ($line in $XMLIn)
        {
            if ($line -match "$ShortDate")
            {
                $line = [regex]::Replace($line, $ShortDate,$($Matches["Shortdate"]))
            }

            if ($line.length -gt 1)
            {
                $line | Out-File -Append -FilePath "$Env:TEMP\$($TaskName).xml"
            }
        }
        
        if ($pscmdlet.ShouldProcess($ComputerName,"Fixing Task: $TaskName "))
        {
            Write-Verbose "Commandline: schtasks /Create /tn $TaskName /XML $Env:TEMP\$($TaskName).xml /f"
            schtasks /Create /tn $TaskName /XML "$Env:TEMP\$($TaskName).xml" /f
        }           

        Write-Verbose "Removing $Env:TEMP\$($TaskName).xml"           

        if (Test-Path "$Env:TEMP\$($TaskName).xml")
        {
            Remove-Item "$Env:TEMP\$($TaskName).xml"
        }
    }
}
```

I have enabled the –WhatIf option again, and have chosen to prompt the user for each task that is to be changed. The script accepts pipeline info, so here again you can pipe results from Get-ScheduledTask into it, and it can be used with parameters to change a single task.

I know there are other ways to access scheduled task information (using the Scheduled Task COM object or WMI), but in my testing I have found the most powerful tool is still the schtasks.exe because it works the same way all the way back to Windows 2000, even though it does have some problems with tasks created in Server 2008/2008 R2.

Download the scripts for this article [here](/downloads/ManagingScheduledTasksInYourEnvironment.zip).
