---
title: Windows Management Framework 4.0 is now available!
author: Shay Levy
type: post
date: 2013-10-25T13:11:56+00:00
url: /2013/10/25/windows-management-framework-4-0-is-now-available/
categories:
  - News
tags:
  - News

---
[Windows Management Framework 4.0][1] is available for installation on Windows 7 SP1, Windows Server 2008 R2 SP1, Windows Server 2012, and Windows Embedded 7.  WMF 4.0 cannot be installed on Windows 8. However, you can obtain the updated functionality included in WMF 4.0 by installing Windows 8.1, which is available as a free update for Windows 8.

WMF 4.0 contains updated versions of the following features:

  * Windows PowerShell
  * Windows PowerShell Integrated Scripting Environment (ISE)
  * Windows PowerShell Web Services (Management OData IIS Extension)
  * Windows Remote Management (WinRM)
  * Windows Management Infrastructure (WMI)
  * Windows PowerShell Desired State Configuration (DSC)

Along with the packages for each operating system, a set of release notes and an additional DSC quick reference is provided. Make sure to read the release notes doc as it contains useful information about new features, as well as a list of known issues with their workarounds and known incompatibilities with other applications. We encourage you to download and read them both.

  * Windows Management Framework 4.0 Release Notes.docx
  * Desired State Configuration Quick Reference for Windows Management Framework 4.0.pdf
  * Desired State Configuration Quick Reference for Windows Management Framework 4.0.pptx

**IMPORTANT:** Not all Microsoft server applications are currently compatible with WMF 4.0. Before installing WMF 4.0, be sure to read the WMF 4.0 Release Notes. Specifically, systems that are running the following server applications should not run WMF 4.0 at this time:

  * System Center 2012 Configuration Manager (not including SP1)
  * System Center Virtual Machine Manager 2008 R2 (including SP1)
  * Microsoft Exchange Server 2013, Microsoft Exchange Server 2010 and Microsoft Exchange Server 2007
  * Microsoft SharePoint 2013 and Microsoft SharePoint 2010
  * Windows Small Business Server 2011 Standard

Microsoft acknowledges that there is still a need for management of Windows Server 2008, and <a href="http://www.microsoft.com/en-us/download/details.aspx?id=34595" target="_blank">Windows Management Framework 3.0</a> remains the answer for Windows Server 2008.

<div>
</div>
Finally, check out these WMF 4.0-related KB articles:

- Description of WMF 4.0 for Windows 7 SP1, Windows Embedded Standard 7 SP1, and Windows Server 2008 R2 SP1 http://support.microsoft.com/kb/2819745

- Description of WMF 4.0 for Windows Server 2012 

  http://support.microsoft.com/kb/2799888

  

- Update is available that prevents the PSModulePath environment variable from being reset when you upgrade WMF 3.0 to WMF 4.0 and then uninstall WMF 4.0 in Windows 

  http://support.microsoft.com/kb/2872047

  

- Update prevents the “PSModulePath” environment variables from being reset after you uninstall WMF 4.0 in Windows http://support.microsoft.com/kb/2872035

[1]: http://www.microsoft.com/en-us/download/details.aspx?id=40855