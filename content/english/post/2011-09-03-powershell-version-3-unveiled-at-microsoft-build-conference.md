---
title: PowerShell Version 3 Unveiled at Microsoft BUILD Conference
author: Keith Hill
type: post
date: 2011-09-03T11:22:37+00:00
url: /2011/09/03/powershell-version-3-unveiled-at-microsoft-build-conference/
views:
  - 4950
post_views_count:
  - 1111
categories:
  - News
tags:
  - News

---
Microsoft took the wraps off Windows PowerShell V3 at the BUILD conference today in the broader context of the Windows Management Framework (WMF).  There were several sessions covering PowerShell today including 10 minutes of the Windows Server 8 keynote where Jeffrey Snover got a chance to talk about the server manageability story with WMF.  Later sessions included:

  * Windows Server 8 apps must run without a GUI &#8211; learn more now
  * Make your product manageable
  * Manage a highly-efficient environment at scale using the Windows Management Framework

During these sessions, Microsoft indicated they invested heavily in:

  * Making it easier to create manageable devices by making it easier to provide CIM data via both native and managed code.  We also heard the WMF would align to CIM standards.
  * Making it easier to manage many machines(physical and virtual) and devices using WMF.  For Windows PowerShell that means, more robust remoting connections including the ability to disconnect and reconnect to remote sessions as well as Microsoft WorkFlow integration to allow such scenarios as allowing one machine to run a workflow remotely on a machine that installs new Windows features, requiring reboots, all within the context of the workflow.  Microsoft also showed off a new Server Dashboard for managing multiple machines.  The UI sports a &#8220;Metro-inspired&#8221; look that is quite elegant.

We also learned that PowerShell V3 would be sitting atop the Dynamic Language Runtime (DLR) and as a result, scripts get compiled allowing them to run up to 6x faster than they would on V2.

PowerShell has beefed up the jobs support in this new version to include an extensibility mechanism referred to as a job source adapter.  If you have something that can be started, stopped, suspended and resumed like a job then you can plug that into the PowerShell job infrastructure such that the existing PowerShell *-Job cmdlets can manipulate it.  A couple of new job cmdlets have been added: Suspend-Job and Resume-Job.  Microsoft has used this mechanism to allow you to create scheduled jobs as well as manage workflows.

And of course, there are lots of new commands.  In fact, there are 44 modules out of the box on the Windows 8 developer preview.  Many of these new modules are provided by various Windows Server teams e.g.:  BranchCache, Dism, DndConfig, DnsLookup, NetAdapter, NetTCPIP, ScheduledTasks, SmbShare, Storage to name a few.

The Windows PowerShell ISE has been updated to provide drop-down Intellisense via an improved tab expansion mechanism.  There are built-in code snippets that you can customize.  And finally, ISE has a recently opened files list!  There are many more new features in PowerShell V3 than I've listed here.  More on that later.

