---
title: Managing scheduled tasks in your environment – Part I
author: Claus Nielsen
type: post
date: 2011-11-21T10:01:27+00:00
url: /2011/11/21/managing-scheduled-tasks-in-your-environment-part-i/
views:
  - 26195
post_views_count:
  - 1883
categories:
  - How To
tags:
  - How To
  - Scheduled Tasks

---
In Part 1 of the series, I will look at getting information on the scheduled tasks running in your environment. One of the things I see a lot in our company is that a lot of scheduled tasks get created on different servers, for one reason or another, but very often these tasks are quickly forgotten.

We kept seeing files show up in a directory they were not supposed to be in, and when we deleted them, they magically reappear after some time.  That led me to believe that it was a scheduled task that was the culprit. So I thought, it was time to get an overview of what scheduled tasks were running in our environment. I had to figure out how to query all the servers in our environment and enumerate enabled scheduled tasks.

I looked at several different ways of getting this information, but in the end I settled on using the built-in schtasks.exe utility and wrapping it in PowerShell function for automation. I started looking at the command line switches for schtasks.exe.  I experimented with the different output formats until I noticed that there was an option to output in CSV format, and as you may already know PowerShell has built-in cmdlets  for working with CSV files. So, I ended up doing:

```powershell
PS> schtasks.exe /query /V /FO CSV | ConvertFrom-Csv
```

This returned a collection of PSCustomObjects that contained information about all the scheduled tasks on my local system. This is all for a single system, but we have 200+ servers, so some more automation was required. The schtasks.exe also supports getting information from remote machines using the /s switch, so that made it rather easy to create a loop that goes through a list of servers. The schtasks.exe works against down-level operating systems back to Windows 2000.

Another thing I had to consider was how to design my script and what data I should return. Should I return all the information schtasks.exe /query /V /FO CSV gave me, or just a subset of it? I decided to return all 28 properties of each scheduled task.  With 28 properties, I quickly decided not to manually create a PSObject, and use Add-Member to populate its properties, but to use a new feature native to PowerShell v2 —the option to supply a hash table of properties to the New-Object cmdlet. To simplify the example, I assume that a list containing server names are available and is named servers.txt.

```powershell
function Get-ScheduledTask
{
    [CmdletBinding()]
    param
    (
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [String[]]
        $ComputerName,
    
        [Parameter(Mandatory = $false)]
        [String[]]
        $RunAsUser,

        [Parameter(Mandatory = $false)]
        [String[]]
        $TaskName,

        [Parameter(Mandatory = $false)]
        [alias("WS")]
        [Switch]
        $WithSpace
    )

    Begin
    {
        $Script:Tasks = @()
    }

    Process {
        $schtask = schtasks.exe /query /s $ComputerName /V /FO CSV | ConvertFrom-Csv
        Write-Verbose "Getting scheduled Tasks from: $ComputerName"

        if ($schtask)
        {
            foreach($sch in $schtask)
            {
                if ($sch."Run As User" -match "$($RunAsUser)" -and $sch.TaskName -match "$($TaskName)")
                {
                    Write-Verbose "$Computername ($sch.TaskName).replace('\','') $sch.'Run As User'"
                    $sch | Get-Member -MemberType Properties | ForEach -Begin { $hash = @{}} -Process {
                        if ($WithSpace)
                        {
                            ($hash.($_.Name)) = $sch.($_.Name)
                        }
                        else
                        {
                            ($hash.($($_.Name).replace(" ",""))) = $sch.($_.Name)
                        }
                    } -End {
                        $Script:Tasks += (New-Object -TypeName PSObject -Property $hash)
                    }
                }
            }
        }
    }

    End {
        $Script:Tasks
    }
}
```

The script accepts pipeline input, so you can do something like:

```powershell
PS> Get-Content c:\servers.txt | Get-ScheduledTask | Out-GridView
```

If you want  to get only scheduled tasks that are run under the Administrator account you could do something like this:

```powershell
PS> Get-Content c:\servers.txt | Get-ScheduledTask –RunAsUser administrator | Out-GridView
```


The above example will list all scheduled tasks running under an account containing the word “Administrator”.  If you want to get only scheduled tasks where the task name contains “FileXfer”:

```powershell
PS> Get-Content c:\servers.txt | Get-ScheduledTask –TaskName FileXfer | Out-GridView
```

You can of course also combine the two:

```powershell
PS> Get-Content c:\servers.txt | Get-ScheduledTask –TaskName FileXfer –RunAsUser administrator
```


A few things to notice though is that since the properties that are returned are generated “on-the-fly” using the names used by schtasks.exe, some of them contain spaces.  Property names should almost never contain spaces, so by default I remove all spaces in the properties, before I create the hash table. A property returned from schtasks.exe called “Run As User” ends up being RunAsUser instead. I have added an option to keep the spaces by supplying the parameter -WithSpace, it will make your output easier to read, if you for instance pipe it to Out-GridView

```powershell
PS> Get-Content c:\servers.txt | Get-ScheduledTask | Select TaskName "RunAsUser"
```

In the above examples the –RunAsUser parameter looks at the property called “RunAsUser”. But it is only called that in the English versions of the operating system.  So, if you are not running this on a machine running English version of the operating system, you will need to manually modify the script to your local locale.

Last but not least, both the -RunAsUser and -TaskName parameters use regular expressions to match a user and a task name, so, for instance, if you are looking for a scheduled tasks called like CopyJob1, CopyJob2 you can do:

```powershell
PS> Get-ScheduledTask –TaskName CopyJob\d  #\d is regex for a single digit
```

Regular expressions are a whole book for themselves, so I will not go into details on it here.  Now we have seen how to collect information about scheduled tasks in your environment, but what if you want to change something on existing scheduled tasks? As I mentioned earlier, we tend to see quite a few scheduled tasks in our environment, and sometimes they get created to run under admin accounts (The quick and dirty way of getting things done).

Our company policy states that admin accounts have to have their passwords changed every 3 months. So we looked for a way to automate those changes as well.