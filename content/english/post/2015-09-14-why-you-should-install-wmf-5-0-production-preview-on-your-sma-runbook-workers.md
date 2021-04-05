---
title: Why you should install WMF 5.0 Production Preview on your SMA runbook workers
author: Ben Gelens
type: post
date: 2015-09-14T16:00:47+00:00
url: /2015/09/14/why-you-should-install-wmf-5-0-production-preview-on-your-sma-runbook-workers/
views:
  - 12429
post_views_count:
  - 1623
categories:
  - Service Management Automation
tags:
  - Service Management Automation

---
Although it is not officially supported yet, here is why you should install <a href="http://t.co/IV0K2ac7Nt" target="_blank">WMF 5.0 Production Preview</a> on your runbook workers right now!

### Background

Because we don’t have native PowerShell support in SMA (yet), the InlineScript activity is used to execute all PowerShell code which cannot be handled by PowerShell Workflow natively. This even may occur without you knowing about it as PowerShell Workflow also does this under the covers for you automatically. It wraps your non-PS workflow commands in an InlineScript activity on a per command basis.

Let’s look at a simple example:

```powershell
workflow test {
    Get-VM | Stop-VM
    Get-VMSwitch
}
```


When loading the workflow in memory, you can investigate the XAML definition.

![](/images/sma1.png)

As you can see, the Get-VM | Stop-VM pipeline is broken up into 2 InlineScripts and the Get-VMSwitch into another one without us specifying this.

The InlineScript activity, when specified explicitly, supports remoting. A functionality I use all the time when dealing with SMA runbooks. Let’s look at an example:


```powershell
workflow StartVM {
    $HVCred = Get-AutomationPSCredential -Name 'HVCredentials'
    InlineScript {
        Get-VM | Where-Object -FilterScript {$_.State -ne 'Running'} | Start-VM
    } -PSComputerName 'HV01' -PSCredential $HVCred -PSAuthentication CredSSP -PSRequiredModules 'Hyper-V'
}
```
Not the most useful example but it serves its purpose. As you can see, the InlineScript activity has the PSComputerName, PSCredential, and PSAuthentication parameters which are used for remoting. This code is actually run on the computer HV01 and not on the runbook worker.

### The InlineScript activity bug

If you run multiple InlineScript activities within a certain amount of time and these connect to the same computer, an ugly bug can show its head.

Consider the following example:


```powershell
workflow Test {
    $MyVar = 'Just some Text'
    InlineScript {
        $ErrorActionPreference = 'Stop'
        Write-Output -InputObject $using:MyVar
    } -PSComputerName 'Hyper-v'
    $MyVar = 'Some other text'
    InlineScript {
        $ErrorActionPreference = 'Stop'
        Write-Output -InputObject $using:MyVar
    } -PSComputerName 'Hyper-v'
}
```
In this example there is no specific reason to specify ErrorActionPreference as stop but this invokes the bug. When run it you would expect that two lines will be shown: ‘Just some Text’ and ‘Some other text’. However, it will output ‘Just some Text’ 2 times. The bug here has broken the Using scope functionality!

![](/images/sma2.png)

Thing gets even worse. Let’s look at another example:


```powershell
workflow Test {
    $MyVar = 'Just some Text'
    InlineScript {
        $ErrorActionPreference = 'Stop'
        Write-Output -InputObject $using:MyVar
    } -PSComputerName 'Hyper-v'
    $SomeOtherVar = 'THIS WILL BE NULL instead off this text'
    InlineScript {
        $ErrorActionPreference = 'Stop'
        Write-Output -InputObject "A completely new variable: $using:SomeOtherVar"
        if ($null -eq $using:SomeOtherVar) {
            'I have nothing!'
        }
    } -PSComputerName 'Hyper-v'
}
```
In this case, in the second InlineScript activity, targets another workflow scope defined variable entirely. You would expect it to output **‘A completely new variable: THIS WILL BE NULL instead of this text’** but it does not contain the variable’s text. Instead, nothing is passed. It is NULL and that’s why we have the **‘I have nothing!’** on screen.

![](/images/sma3.png)

Since SMA uses PowerShell Workflow, the bug is invoked here as well. My friend and fellow InlineScript sufferer <a href="https://twitter.com/trondhindenes" target="_blank">Trond Hindenes</a> has logged this bug on <a href="https://connect.microsoft.com/PowerShell/feedback/details/894721/workflows-using-multiple-inlinescripts-with-the-pscomputername-does-not-get-correct-variables-from-the-workflow-when-using-using-scoping" target="_blank">Connect</a>.

### WMF 5.0 production preview to the rescue!

The reason why you should install the WMF 5.0 Production Preview package is: The PowerShell team solved the issue!

Please note that SMA does not officially support WMF 5.0 yet but I’ve heard from good sources that Microsoft is aware people are using the combination successfully in production already without known issues.

Sample runbook:


```powershell
workflow WMF5Test {
    $TestCred = Get-AutomationPSCredential -Name 'TestCredentials'
    Write-Output -InputObject "Handled on Runbook Worker $($env:ComputerName) with PSVersion $($psversiontable.PSVersion.Major)"
#region first run
$PassingVar = '1'

InlineScript {
    $ErrorActionPreference = 'Stop'
    Write-Output -InputObject "First Run. Var value: $using:PassingVar Running on $($env:ComputerName) with PSVersion $($psversiontable.PSVersion.Major)"
} -PSComputerName 'Server01' -PSCredential $TestCred -PSAuthentication CredSSP -PSUseSsl $true

InlineScript {
    $ErrorActionPreference = 'Stop'
    Write-Output -InputObject "First Run. Var value: $using:PassingVar Running on $($env:ComputerName) with PSVersion $($psversiontable.PSVersion.Major)"
} -PSComputerName 'Server02' -PSCredential $TestCred -PSAuthentication CredSSP
#endregion first run

#region second run
$PassingVar = '2' #overwriting variable for second run
$AditionalVar = 'A String' #add aditional Var to second run output

InlineScript {
    $ErrorActionPreference = 'Stop'
    Write-Output -InputObject "Second Run. Var value: $using:PassingVar Running on $($env:ComputerName) with PSVersion $($psversiontable.PSVersion.Major). Second Var: $using:AditionalVar"
} -PSComputerName 'Server01' -PSCredential $TestCred -PSAuthentication CredSSP -PSUseSsl $true

InlineScript {
    $ErrorActionPreference = 'Stop'
    Write-Output -InputObject "Second Run. Var value: $using:PassingVar Running on $($env:ComputerName) with PSVersion $($psversiontable.PSVersion.Major) Second Var: $using:AditionalVar"
} -PSComputerName 'Server02' -PSCredential $TestCred -PSAuthentication CredSSP
#endregion second run
}
```
Results when the sample runbook is run through a runbook worker still on WMF 4.0:

![](/images/sma4.png)

Results when the sample runbook is run through a runbook worker on WMF 5.0:

![](/images/sma5.png)

As you can see, the installation of WMF 5.0 would only need to occur on the runbook workers themselves for this bug to be resolved. This makes sense as the XAML is compiled here. When the runbook workers are updated, both WMF 4.0 and WMF 5.0 targets get the values passed correctly!