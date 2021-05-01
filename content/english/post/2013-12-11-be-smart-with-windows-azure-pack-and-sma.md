---
title: Be “SMART” with Windows Azure Pack and SMA
author: Jim Britt
type: post
date: 2013-12-11T19:00:09+00:00
toc: true
url: /2013/12/11/be-smart-with-windows-azure-pack-and-sma/
categories:
  - How To
  - Azure Pack
tags:
  - How To
  - Azure Pack

---
### Service Management Automation (SMA)

So, what exactly is SMA and why is it important in the world of PowerShell?  I’ll start with a link where you can read even more information at your leisure <http://aka.ms/IntroToSMA>.  This link starts off with the basics and then introduces you to some of the concepts.  However, to give you a general idea, SMA is an automation engine that ships with System Center Orchestrator that is leveraged by the **Windows Azure Pack** (WAP) built entirely on Windows PowerShell Workflow. Think a Silverlight UI that looks like Azure with PowerShell Workflow under the hood leveraged for automation of that solution.

![](/images/smasmart1.png)

### SMA Runbook Toolkit (SMART) for Runbook Import and Export

Now that we’ve gotten the formalities out of the way, I wanted to dive into some specifics around a new solution I put together for SMA that will streamline the process of sharing PowerShell Workflows (SMA Runbooks).  True, this solution directly leverages SMA, so in order to really appreciate it, you’ll need to dive into the intro above and get your hands on the solution.  However, it may pique your interest enough to investigate it a bit and see the power of the solution [then I’ve done my job too J &#8211; after all I want you to do more than just read about it – “kick the tires” and let me know what you think]. You can read in depth about it here: [**Automation–Service Management Automation–SMA Runbook Toolkit Spotlight–SMART for Runbook Import and Export**][1] but I’m also going to give you the basic details in summary within this article so please read on!

#### Solution Breakdown

  * A set of SMA Runbooks and PowerShell Workflow scripts to assist in the import and export of SMA Runbooks into and out of SMA
  * Provides a mechanism to encapsulate a set of Runbooks and properties around those Runbooks into a compilation of atomic XML files that create a portable and easily sharable solution between environments
  * Can facilitate source backup into a repository on a scheduled basis to ensure SMA Runbooks are backed up on a regular basis (with a broad range of properties that encompass those Runbooks)
  * Streamlines the process of automated import and export beyond the provided PowerShell cmdlets that come with the SMA solution
  * Boolean parameters allow for flexible management of property export and import (you choose what you want to import and export and how)

#### Contents of the Solution Download

![](/images/smasmart1.png)

The SMART for Runbook Import and Export comes with a series of PowerShell scripts, XML files (atomic SMA Runbooks), and a User Guide to get you started.

  * **Install-SMARTForRunbookImportAndExport.ps1:**  Installation wrapper to get the solution installed within your SMA environment&#8211;right click and “Run with PowerShell”. The result will be (**4**) Runbooks imported into your SMA environment that include tags for easy searching.

  * **Export-SMARunbookToXML**: This is essentially the Runbook version of the PowerShell version Export-SMARunbookToXML.ps1

    ![](/images/smasmart3.png)

  * **Import-SMARunbookfromXMLorPS1**: This is the Runbook version of Import-SMARunbookfromXMLorPS1.ps1

    ![](/images/smasmart4.png)

  * **Invoke-SMARunbookExport**: This Runbook provides a wrapper framework for executing an SMA Runbook Export
    ![](/images/smasmart5.png)
  * **Invoke-SMARunbookImport**: This Runbook provides a wrapper framework for executing an SMA Runbook Import
    ![](/images/smasmart6.png)

### Summary – Closing Remarks

In closing, I want to thank you for checking out this article and providing any feedback or comments to the solution I’ve covered here.  I definitely recommend taking a closer look at the following supporting blog post that also provides the direct download for this PowerShell-based solution.  [**Automation–Service Management Automation–SMA Runbook Toolkit Spotlight–SMART for Runbook Import and Export**][1] and _Happy Automating!_ J

[1]: http://blogs.technet.com/b/privatecloud/archive/2013/10/23/automation-service-management-automation-sma-runbook-toolkit-spotlight-smart-for-runbook-import-and-export.aspx