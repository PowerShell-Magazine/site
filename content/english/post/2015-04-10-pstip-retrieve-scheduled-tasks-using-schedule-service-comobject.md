---
title: 'PSTip: Retrieve scheduled tasks using Schedule.Service COMObject'
author: Jaap Brasser
type: post
date: 2015-04-10T18:00:43+00:00
url: /2015/04/10/pstip-retrieve-scheduled-tasks-using-schedule-service-comobject/
views:
  - 23740
post_views_count:
  - 7052
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In PowerShell 4.0, the Get-ScheduledTask cmdlet was introduced, but unfortunately this cmdlet is not available in older versions of PowerShell. This is where the Schedule.Service COMObject can be useful to enumerate scheduled tasks on a local or remote system. Unfortunately, unlike the Get-ScheduledTask cmdlet using this COMObject requires an elevated PowerShell console with administrative credentials. The following example will list the all tasks in the \ folder of the Task Scheduler and display the Name, Path and State of the tasks:

```powershell
$Schedule = New-Object -ComObject "Schedule.Service"
$Schedule.Connect('localhost')
$Folder = $Schedule.GetFolder('\')
$Folder.GetTasks(1) | Select Name,Path,State
```


Unfortunately this outputs the state as an integer instead of a string. To resolve this calculated properties in combination with the switch operator will display the user-friendly names:

```powershell
$Schedule = New-Object -ComObject "Schedule.Service"
$Schedule.Connect('localhost')
$Folder = $Schedule.GetFolder('\')
$Folder.GetTasks(1) | Select Name,Path,@{
    Name = 'State'
    Expression = {switch ($_.State) {
            0 {'Unknown'}
            1 {'Disabled'}
            2 {'Queued'}
            3 {'Ready'}
            4 {'Running'}
        }
    }
}
```


This still provides only the scheduled tasks that are listed in the root folder. In order to recursively search through all folders additional code will be required. A script that iterates through all folders in the scheduler and displays all tasks is available in the TechNet Script Gallery: [Get-ScheduledTask][1]

```powershell
param(
	$computername = "localhost",
    [switch]$RootFolder
)

#region Functions
function Get-AllTaskSubFolders {
    [cmdletbinding()]
    param (

        # Set to use $Schedule as default parameter so it automatically list all files

        # For current schedule object if it exists.

​        $FolderRef = $Schedule.getfolder("\")
​    )
​    if ($FolderRef.Path -eq '\') {
​        $FolderRef
​    }
​    if (-not $RootFolder) {
​        $ArrFolders = @()
​        if(($folders = $folderRef.getfolders(1))) {
​            $folders | ForEach-Object {
​                $ArrFolders += $_
​                if($_.getfolders(1)) {
​                    Get-AllTaskSubFolders -FolderRef $_
​                }
​            }
​        }
​        $ArrFolders
​    }
}
#endregion Functions

try {
	$schedule = New-Object –ComObject ("Schedule.Service")
} catch {
	Write-Warning "Schedule.Service COM Object not found, this script requires this object"
	return
}

$Schedule.connect($ComputerName)
$AllFolders = Get-AllTaskSubFolders

foreach ($Folder in $AllFolders) {
    if (($Tasks = $Folder.GetTasks(1))) {
        $Tasks | Foreach-Object {
	        New-Object -TypeName PSCustomObject -Property @{
	          'Name' = $_.name
                'Path' = $_.path
                'State' = switch ($_.State) {
                    0 {'Unknown'}
                    1 {'Disabled'}
                    2 {'Queued'}
                    3 {'Ready'}
                    4 {'Running'}
                    Default {'Unknown'}
                }
                'Enabled' = $_.enabled
                'LastRunTime' = $_.lastruntime
                'LastTaskResult' = $_.lasttaskresult
                'NumberOfMissedRuns' = $_.numberofmissedruns
                'NextRunTime' = $_.nextruntime
                'Author' =  ($_.xml).Task.RegistrationInfo.Author
                'UserId' = ($_.xml).Task.Principals.Principal.UserID
                'Description' = ($_.xml).Task.RegistrationInfo.Description
            }
        }
    }
}
```

[1]: https://gallery.technet.microsoft.com/scriptcenter/Get-Scheduled-tasks-from-3a377294