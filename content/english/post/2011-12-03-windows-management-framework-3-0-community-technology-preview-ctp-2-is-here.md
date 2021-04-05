---
title: Windows Management Framework 3.0 Community Technology Preview (CTP) 2 is here
author: Shay Levy
type: post
date: 2011-12-03T12:19:33+00:00
url: /2011/12/03/windows-management-framework-3-0-community-technology-preview-ctp-2-is-here/
views:
  - 5812
post_views_count:
  - 918
categories:
  - News
tags:
  - News

---
The PowerShell team has [announced the release ][1] of Windows Management Framework 3.0 CTP2. The new build contains Windows PowerShell 3.0, WMI and WinRM, and is available to be installed on Windows 7 SP1 and Windows Server 2008 R2 SP1.

**IMPORTANT:** If you have WMF3.0 CTP1 installed, you must uninstall it before installing CTP2.

### Overview of changes since WMF 3.0 CTP1

1. **Customer Reported Bug Fixes**: Many customer reported bugs have been fixed since the WMF 3.0 CTP1. The release notes contains a list of bug titles, but please check [Connect][2] for full details.

2. **Single Command Pane in Windows PowerShell ISE**: The Command and Output panes in Windows PowerShell ISE have been combined into a single Command pane that looks and behaves like the Windows PowerShell console.

3. **Updatable Help:** The WMF 3.0 CTP1 release notes described a new Updatable Help system in Windows PowerShell 3.0 and included a copy of the help content. The Updatable Help system is now active on the Internet. To download and update help files, type: Update-Help.

4. **Windows PowerShell Workflows:** A number of enhancements have been made in the scripting experience for Windows PowerShell Workflows, including new keywords: Parallel, Sequence & Inlinescript. A document describing these changes will be published to the download page shortly.

5. **Remote Get-Module:** The Get-Module cmdlet now supports implicit remoting. You can now use the new PSSession and CIMSession parameters of the Get-Module cmdlet to get the modules in any remote session or CIM session. A number of other module enhancements are listed in the release notes.

### Feedback & Bugs

We welcome any feedback or bug submissions to the Windows PowerShell Connect site: <http://connect.microsoft.com/PowerShell>

### Additional Information:

This software is a pre-release version. Features and behavior are likely to change before the final release. You can download CTP2 [HERE][1].

[1]: http://www.microsoft.com/download/en/details.aspx?id=27548
[2]: http://connect.microsoft.com/powershell