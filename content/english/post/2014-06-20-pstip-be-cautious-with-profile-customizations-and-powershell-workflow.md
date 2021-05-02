---
title: '#PSTip Be cautious with profile customizations and PowerShell Workflow'
author: Jan Egil Ring
type: post
date: 2014-06-20T18:00:49+00:00
url: /2014/06/20/pstip-be-cautious-with-profile-customizations-and-powershell-workflow/
categories:
  - Service Management Automation
  - Tips and Tricks
tags:
  - Service Management Automation
  - Tips and Tricks

---
When working with Windows PowerShell Workflow, there are things to be aware of which can break workflow from working. One such thing is customizations made in the PowerShell profile.

Let&#8217;s start by defining an example workflow we can work with:

```
Workflow Get-RemoteService {
	Get-Service -PSComputerName $PSComputerName
}
```

When invoking the workflow I got the following errors on my computer:

```
PS [3]: Get-RemoteService -PSComputerName srv1

WARNING: The binding of default value &#8216;True&#8217; to parameter 'Keep' failed: Cannot bind parameter &#8216;Keep&#8217; to the target. Exception setting "Keep The Wait" and "Keep parameters cannot be used together in the same command." 

Cannot find drive. A drive with the name 'Scripts' does not exist.
+ CategoryInfo : ObjectNotFound: (Scripts:String) [], ParentContainsErrorRecordException
+ FullyQualifiedErrorId : DriveNotFound
+ PSComputerName : [localhost]
What is going on here? Let\`s start by looking at the warning: Cannot bind parameter 'Keep' to the target.
```

This one is hard to track down without knowing that workflow actually uses PowerShell jobs under the hood. The -Wait and -Keep parameters mentioned in the warning message is available on the Receive-Job cmdlet. The two parameters belongs to different parameter sets, and can not be used together. In my case I had the following defined in my PowerShell profile:

```
$PSDefaultParameterValues.Add("Receive-Job:Keep",$True)
```


To verify this was causing the problem I removed Receive-Job from $PSDefaultParameterValues and re-ran the workflow:

```
$PSDefaultParameterValues.Remove("Receive-Job:Keep")
Get-RemoteService -PSComputerName srv1
```


This resolved the warning:

```
PS [5]: Get-RemoteService -PSComputerName srv1

Cannot find drive. A drive with the name 'Scripts' does not exist.
+ CategoryInfo : ObjectNotFound: (Scripts:String) [], ParentContainsErrorRecordException
+ FullyQualifiedErrorId : DriveNotFound
+ PSComputerName : [localhost]
```

For the second error, we can see an error regarding the Scripts: drive. This is a PSDrive defined in my PowerShell profile, which is the current location in my PowerShell session. Since workflow runs in it\`s own runspace, the drive cannot be found.

After changing the current directory to C: which is also available to the workflow runspace, the workflow runs without issues.

```
cd c:
Get-RemoteService -PSComputerName srv1
```


The easiest way to exclude profile customizations from breaking a workflow is by launching powershell.exe/powershell_ise.exe with the -NoProfile switch. If the workflow then runs without issues, you should start looking at your PowerShell profile to see what might be interfering with workflow.