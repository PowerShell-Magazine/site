---
title: Rally the orphan processes
author: Robert Robelo
type: post
date: 2011-09-22T13:34:26+00:00
url: /2011/09/22/rally-the-orphan-processes/
views:
  - 11603
post_views_count:
  - 2433
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Some processes start child processes; they are set up that way. Sometimes, when the parent process stops, or it is stopped, some of the child processes remain running. That can be by design, but in some cases the _orphan_ process needs to be terminated as well.

The first step in determining if a running process is an orphan process is to check if it has a parent process. Next, we need to check if the parent process is running or not; this can cause a problem though (more on this later). If the parent process is not running, we got ourselves an orphan process.

First things first, let's determine if a process has a parent process. It is not easy to achieve this task with the Get-Process cmdlet because the System.Diagnostics.Process object does not have a Parent* property. That is fine—we can take the Windows Management Instrumentation (WMI) route. The Win32_Process class is used to retrieve running processes, we just need to pass it to the Get-WmiObject <span style="font-family: Arial;">cmdlet. The objects returned do have a ParentProcessID property, exactly what we are looking for.</span>

Next, we have to resolve the parent process&#8217; state, whether it is running or not. The process of interest&#8217;s ParentProcessID value can also help us with this task, but it will not be a straight forward solution. The problem arises if the parent process is not running and its ID, stored in the potentially <em>orphaned</em> process, was assigned to a new process that is not related to the process of interest. To ensure that the process that has the ParentProcessID as its ID is the parent, we will compare its creation time against that of our process of interest. If our process was started—or created—after the possible parent process creation time, the parent is legitimate, it is not a foster parent.

Creating the filter we will apply to all running process sounds complicated but it is easy to understand. Basically, we will ask PowerShell to filter through the piped process object if…

  * it has a ParentProcessID but its parent process is not running, or
  * its creation time is less than the prospective parent process&#8217; creation time, that is, it started before the substitute parent

Finally, any process object that goes through the filter is piped and bound to the Get-Process cmdlet via its ProcessID value. This way we have the list of orphan System.Diagnostics.Process objects.

Here is the Get-OrphanProcess script:

```powershell
# WMI class instace to convert WMI time to DateTime
$wmi = [WMI]

# retrieve and filter running processes
Get-WmiObject Win32_Process | Where-Object {
    # attempt to retrieve the piped process' parent process
    $parent = Get-WmiObject Win32_Process -Filter "ProcessID='$($_.ParentProcessID)'"

    # get the piped process, as well as its parent's, creation time
    $creationDate, $parentCreationDate = $(
        # if piped process has a parent and its parent is running
        if ($_.ParentProcessID -and $parent)
        {
            # convert their the WMI creation time to DateTime
            $wmi.ConvertToDateTime($_.CreationDate),$wmi.ConvertToDateTime($parent.CreationDate)

        }
        else
        {
            # return Null
            $null, $null
        }
    )
    
    # filter piped process through if its parent process is not running or
    # its creation time happened before the parent process was started
    -not $parent -or $creationDate -lt $parentCreationDate

    # pipe the filtered process to Get-Process binding it via its ProcessID
} | Get-Process -ID { $_.ProcessID}

# clean up
Remove-Variable wmi, parent,creationDate, parentCreationDate
```

Now that we have a list of <em>the</em> <em>orphan ones</em>, we can filter further and do with them what we find appropriate. To kill or not to kill, that is the question.

Let me demonstrate. Let&#8217;s suppose the Get-OrphanProcess.ps1 script is located in C:\Scripts (it could be anywhere, really, just change the call to the script in the sample code accordingly); we will use it to retrieve an <em>orphan calc process</em> we will start from a non-interactive PowerShell session that will also output its process ID before it exits. Here is the code and the output it returned. Try it!

```powershell
& {
    $VerbosePreference = 'Continue'
    Stop-Process -Name calc -ErrorAction SilentlyContinue
    $parentProcessId = PowerShell -NoProfile -NonInteractive -Command {$pid; Start-Process -FilePath calc.exe}
    $orphanCalcId = (Get-Process -Name calc).Id
    $orphanCalcParentId = (Get-WmiObject Win32_Process -Filter "ProcessID = ''$orphanCalcId'").ParentProcessID
    Write-Verbose "Parent or spawner process ID: $parentProcessId"
    Write-Verbose "Orphan calc's parent process ID: $orphanCalcParentId"
    Write-Verbose "Are they equal?’ $($parentProcessId -eq $orphanCalcParentId)"
    Write-Verbose "Orphan calc's process ID: $orphanCalcId"
    Write-Verbose 'Starting another calc process'
    Start-Process -FilePath calc.exe
    Write-Verbose 'Here are the running calc processes. One is an orphan process.'
    Get-Process -Name calc
    Write-Verbose 'Here is the orphan calc process.'
    C:\Scripts\Get-OrphanProcess.ps1 | Where-Object {$_.ProcessName -eq 'calc'}
}

VERBOSE: Parent or spawner process ID: 7448
VERBOSE: Orphan calc's parent process ID: 7448
VERBOSE: Are they equal?' True
VERBOSE: Orphan calc's process ID: 2312
VERBOSE: Starting another calc process
VERBOSE: Here are the running calc processes. One is an orphan process.
Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id ProcessName
-------  ------    -----      ----- -----   ------     -- -----------
      69      14     6860      12132    85     0.03   2312 calc      
      38       6     1584       3388    60     0.02   3116 calc 
      
VERBOSE: Here is the orphan calc process.
      74      19     7344      13628    83     0.05   2312 calc
```

[1]: http://technet.microsoft.com/en-us/library/dd347630.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.diagnostics.process.aspx
[3]: http://msdn.microsoft.com/en-us/library/aa394372(VS.85).aspx
[4]: http://technet.microsoft.com/en-us/library/dd315295.aspx