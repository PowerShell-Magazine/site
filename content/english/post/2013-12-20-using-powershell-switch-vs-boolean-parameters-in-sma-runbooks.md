---
title: Using PowerShell Switch vs. Boolean Parameters in SMA Runbooks
author: Jim Britt
type: post
date: 2013-12-20T17:00:50+00:00
url: /2013/12/20/using-powershell-switch-vs-boolean-parameters-in-sma-runbooks/
categories:
  - How To
  - Service Management Automation
tags:
  - How To
  - Service Management Automation

---
Hey Readers.  In this article, I wanted to take the opportunity to talk about a scenario that may be “edge case” but a case just the same within the constructs of leveraging switch parameters in PowerShell workflow.  In fact, this scenario becomes less of an edge case as you start to write SMA Runbooks that leverage nested Runbooks.  Still don’t know what SMA is and want more information?  Check out the series <http://aka.ms/IntroToSMA> and my first article for PowerShell Magazine [here](/2013/12/11/be-smart-with-windows-azure-pack-and-sma/) for more information.

### Scenario

For me, a journey started with PowerShell workflow as I started diving into SMA (Service Management Automation). SMA leverages PowerShell workflow under the hood for its automation engine.  Due to the nature of how SMA uses inline Runbooks (PowerShell workflow) to execute tasks and return back to the main calling Runbook, certain behaviors exist that may be unexpected. In this scenario, the switch parameter capabilities get somewhat lost and your approach will need to change to accommodate.  Here are some examples to illustrate.

#### Example 1

Executing a PowerShell workflow directly specifying a switch parameter (not nested)

```
Workflow Invoke-NestedWorkflow
{
    Param([switch]$SomeSwitch)
    $SomeSwitch.IsPresent
}

Invoke-NestedWorkflow -SomeSwitch
```

![](/images/smabool1.png)

A switch parameter was sent that should indicate a value of _True_ and in fact was validated as _True_. No surprises.

### Example 2

To mimic a nested Runbook in SMA, I&#8217;ve combined two workflows and am leveraging a switch parameter.

```
Workflow Invoke-NestedWorkflow
{
    Param([switch]$SomeSwitch)
    "I'm in the nested Runbook"
    $SomeSwitch.IsPresent
}

Workflow Invoke-ParentWorkflow
{
    "I'm in the parent Runbook"
    Invoke-NestedWorkFlow -SomeSwitch
}

Invoke-ParentWorkflow
```

![](/images/smabool2.png)

This time you'll notice that as we send _-SomeSwitch_ into the nested workflow, it is not interpreted properly and we receive an error message. [PowerShell v4] 

Same script [PowerShell v3 below]

![](/images/smabool3.png)

**Note**  Different results between versions of PowerShell but error(s) still thrown.

### Example 3

In order to retain the switch parameter, you have to do some extra work and it isn’t necessarily as clean as a native PowerShell switch parameter usually is.  Specifying _**-SomeSwitch:****$True**_ now allows you to retain the _“$True”_ through the execution on this switch parameter.  But it isn’t pretty :).

```
Workflow Invoke-NestedWorkflow
{
    Param([switch]$SomeSwitch)
    "I'm in the nested Runbook"
    $SomeSwitch.IsPresent
}

Workflow Invoke-ParentWorkflow
{
    "I'm in the parent Runbook"
    Invoke-NestedWorkFlow -SomeSwitch:$true
}

Invoke-ParentWorkflow
```

(no errors and _True_ retained)

![](/images/smabool4.png)

[Recommendation]

Why not use Boolean?  We’re nearly there anyway at this point.  See below, the call to the parent Runbook is now using _-SomeSwitch $True_. Instead of _-SomeSwitch:$True_ and the parameter types have been updated to [bool].

```
Workflow Invoke-NestedWorkflow
{
    Param([bool]$SomeSwitch)
    "I'm in the nested Runbook"
    $SomeSwitch
}

Workflow Invoke-ParentWorkflow
{
    "I'm in the parent Runbook"
    Invoke-NestedWorkFlow -SomeSwitch $true
}

Invoke-ParentWorkflow
```

> -SomeSwitch is now a _Boolean_ parameter and should be renamed of course, but left “as is” for consistency.

![](/images/smabool5.png)

### Summary and Takeaways

So in summary, switch parameters used in SMA (and nested PowerShell workflow) can still be used but require some modifications to function properly.  Taking a closer look, Booleans fit nicely here and take on their native look at feel so why not use them instead!  Want to get a bit more information on where this topic came from?  Check out this blog post [Automation–Service Management Automation–SMA Runbook Toolkit Spotlight–SMART for Runbook Import and Export][1] on the [Building Clouds][2] blog that triggered this topic where [Aleksandar Nikolic][3] and me debated on this subject which ended up driving quality and awareness around SMA and PowerShell workflow!

Till next time …

Happy Automating!

[1]: http://blogs.technet.com/b/privatecloud/archive/2013/10/23/automation-service-management-automation-sma-runbook-toolkit-spotlight-smart-for-runbook-import-and-export.aspx
[2]: http://aka.ms/BuildingClouds
[3]: http://104.131.21.239/author/aleksandar/