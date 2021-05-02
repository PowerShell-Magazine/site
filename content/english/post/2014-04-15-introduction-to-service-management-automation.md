---
title: Introduction to Service Management Automation
author: Jan Egil Ring
type: post
date: 2014-04-15T16:00:39+00:00
url: /2014/04/15/introduction-to-service-management-automation/
categories:
  - Service Management Automation
tags:
  - Service Management Automation

---
Service Management Automation (SMA) is a new component in System Center for process automation.

Before looking into the technical details, let&#8217;s have a look at what practical use cases there are for an automation product like SMA:

  * Automate **maintenance tasks** such as orchestrating patching of Windows Failover clusters, stop and start services as well as starting and stopping virtual machines in a defined order when doing maintenance work
  * Automate **business processes** such as creating, changing and removing user accounts, password resets and automating file transfers
  * Automating tasks in a **service catalog and change management** such as creating virtual machines and automatic configuration of backups
  * **Dynamic resource allocation** based on load or calendar. A practical example might be extending a web farm with additional instances before the Christmas holidays
  * **Respond to alarms** from operations tools such as System Center Operations Manager
  * **Integrations across systems**, such as automatically creating an incident in an incident management system based on an event from an operations system

SMA is not a separate component in the System Center suite on the same level as the other components such as Configuration Manager and Virtual Machine Manager; the installation files are included in the installation media for Orchestrator:

![](/images/sma001.png)

As we can see based on the above screenshot, SMA has 3 components:

<b style="line-height: 1.5em;">Web Service</b> – Web API (RESTful OData) used for several tasks&#8211;communication with Windows Azure Pack, distribution of runbook jobs to Runbook Workers, as well as delegating permissions.

**Runbook Worker** – Executes runbook jobs. These might be scheduled or triggered manually.

<b style="line-height: 1.5em;">PowerShell Module</b> – Cmdlets for administering SMA.

In addition, a SQL database is needed to store data (needs to be specified during installation). From an architecture level there are similarities between System Center Orchestrator (SCO) and Service Management Automation (SMA). The main difference in the user experience is that SCO has a graphical interface with drag-and-drop support, while SMA is 100% based on Windows PowerShell Workflow.

PowerShell Workflow, introduced in PowerShell 3.0, is built upon [Windows Workflow Foundation][1]. PowerShell Workflow makes it possible to orchestrate long-running activities and automate complex tasks, such as deploying a service containing several servers. PowerShell Workflow is capable of executing activities in parallel, resuming execution after a failure (such as network outage) and surviving a restart of a managed node. Details and links to more information about PowerShell Workflow are available in the article «[When Windows PowerShell Met Workflow][2]» on the Windows PowerShell Team blog.

### Portal

SMA itself does not have a built-in portal; the only way to define runbooks is by using the PowerShell cmdlets or the web service. Windows Azure Pack for Windows Server is a free component for hosting providers and enterprises which provides a user experience consistent with Windows Azure:

![](/images/sma002.png)

One of the components of Windows Azure Pack is Automation, which is based on a connection to the SMA web service. This makes it possible to use the Automation component available in Windows Azure Pack as a web portal for SMA:

![](/images/sma003.png)

The first item we see when entering the Automation portal is the _Dashboard_, which provides an overview of the defined runbooks.

_Runbooks_ provides a view of all runbooks defined, with options for sorting based on status, tags or search query:

![](/images/sma004.png)

Clicking on a runbook name brings you to a dedicated dashboard for the selected runbook, where statistics for the runbook jobs are available:

![](/images/sma005.png)

_Jobs_ provides history and output for each job instance, which has been executed for the runbook:

![](/images/sma006.png)

_Author_ shows the published version of the runbook:

![](/images/sma007.png)

_Draft_ provides the ability to edit and test the code defined in the runbook:

![](/images/sma008.png)

When you authoring a runbook, the runbook has to start with the keyword _workflow_ followed by the name of the runbook. Optionally a param block can be provided in order to define parameters for the runbook. The actual code is defined after the param block.

If parameters are defined, values for these may be provided when starting the runbook:

![](/images/sma009.png)

These values can also be provided if the runbook is scheduled.

Schedules may be defined in the next menu item:

![](/images/sma010.png)

![](/images/sma011.png)

The last menu item for a runbook is _Configure_, giving options such as providing a description, tags and enabling debugging and logging:

![](/images/sma012.png)

Debugging and logging will write a lot of data in the database, and should only be enabled during troubleshooting.

The last item we will have a look at in the Automation portal in Windows Azure Pack is _Assets_:

![](/images/sma013.png)

An asset (also called a global resource) may be a PowerShell module or one of the following:

![](/images/sma014.png)

_Connections_ are connections to other systems, such as other System Center components like Data Protection Manager and Virtual Machine Manager. _Credentials_, _variables_, and _certificates_ can be defined in order to avoid hardcoding them in runbooks. The last type of asset is a _schedule_, which we have already looked at. All assets are globally available and shared among all runbooks.

## Architecture

When PowerShell Workflow is used outside of SMA, state («persistence») is stored on a file system on the machine executing the workflow. With SMA we get a highly available workflow, because persistence is stored in a SQL database.

This makes it possible to build a highly available automation platform, by configuring a highly available SQL service (using clustering or AlwaysOn) as well as 2 or more servers with the SMA Runbook Worker and web service installed:

![](/images/sma015.png)

Source of illustration: System Center Orchestrator Engineering Team Blog

At the Build conference in April 2014 Microsoft also [announced][3] Microsoft Azure Automation preview, making it possible to leverage SMA without an on-premises automation platform.

## Summary

Since SMA is based on PowerShell Workflow it&#8217;s possible to automate everything which can be accomplished from PowerShell. Unlike Orchestrator which can be extended by Integration Packs, SMA can be extended using PowerShell modules (they are called Integration Modules).

The first version of the product lacks some functionality available in Orchestrator, such as the ability to configure runbook permissions. This is a feature an enterprise using SMA would find useful in order to delegate access to runbooks based on, for example, Active Directory security groups.

There is no official statement regarding the strategy around co-existence of Orchestrator and SMA in the future, but it&#8217;s not unlikely that SMA will overtake Orchestrator&#8217;s role in the Microsoft ecosystem when more functionality comes in place.

[1]: http://en.wikipedia.org/wiki/Windows_Workflow_Foundation
[2]: http://blogs.msdn.com/b/powershell/archive/2012/03/17/when-windows-powershell-met-workflow.aspx
[3]: http://blogs.technet.com/b/server-cloud/archive/2014/04/03/announcing-microsoft-azure-automation-preview.aspx
[4]: http://technet.microsoft.com/en-us/library/dn469258.aspx
[5]: http://azure.microsoft.com/en-us/documentation/services/automation/?WT.mc_id=Blog_SC_Automation_Azure