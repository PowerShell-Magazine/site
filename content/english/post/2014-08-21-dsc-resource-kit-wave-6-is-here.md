---
title: DSC Resource Kit Wave 6 is here!
author: Ravikanth C
type: post
date: 2014-08-21T09:14:18+00:00
url: /2014/08/21/dsc-resource-kit-wave-6-is-here/
categories:
  - PowerShell DSC
  - News
tags:
  - PowerShell DSC
  - News

---
Windows PowerShell team published a new wave of DSC Resource Kit. It is [Wave 6][1]!

This wave contains four new resources:

  * xChrome &#8211; Helps installing Chrome browser
  * xFirefox &#8211; Helps installing Firefox browser
  * xSafeHarbor &#8211; Installs the Safe Harbor sample
  * xRemoteDesktopAdmin &#8211; Enables management of RDP and Firewall settings for RDP

### Installation

To install all DSC Resource Kit Modules

  * Unzip the content under $env:ProgramFiles\WindowsPowerShell\Modules folder

To confirm installation:

  * Run **Get-DSCResource** to see that all of the resources on this page are among the DSC resources listed

### Requirements

Most items in this release require the latest version of PowerShell (v4.0, which ships in Windows 8.1 or Windows Server 2012 R2). To easily use PowerShell 4.0 on older operating systems, [install WMF 4.0 ][2]. Please read the installation instructions that are present on both the download page and the release notes for WMF 4.0.

Some samples and resources in this release the Windows Management Framework (WMF) 5.0 Experimental Release July 2014, which contains functionality that has been updated from WMF 4.0. Each module with this requirement will be clearly identified. The WMF 5.0 Experimental Release July 2014 is available for installation on Windows 8.1 and Windows Server 2012 R2. More information about the content of the WMF 5.0 Experimental Release July 2014 is available in its dedicated release notes, included in the download links below.

**Notice: **WMF 5.0 Experimental Release July 2014 is delivered as an MSU installation package via the links below. Installing this will update the PowerShell, WMI, and WinRM components of your Windows installation. change the state of your machine, as opposed to the scripts in Resource Kit. If you choose “Open” from either the x64 or x86 direct download links, the package will be downloaded, and the install will update your system with these new components.

**Disclaimer:** There are some scenarios in WMF 5.0 Experimental July 2014 with incomplete or missing functionality. This is also included in the dedicated release notes, which are included in the download links below.

Direct Download Links:

  * [Release Notes][3]
  * [x64 MSU][4]
  * [x86 MSU][5]

[1]: http://gallery.technet.microsoft.com/DSC-Resource-Kit-All-c449312d
[2]: http://www.microsoft.com/en-us/download/details.aspx?id=40855
[3]: http://download.microsoft.com/download/E/D/B/EDB86AD9-4D26-4C33-A8B2-82BE161682E2/Windows%20Management%20Framework%205.0%20Experimental%20Release%20July%202014%20Release%20Notes.docx
[4]: http://download.microsoft.com/download/E/D/B/EDB86AD9-4D26-4C33-A8B2-82BE161682E2/WindowsBlue-KB2969050-x64.msu
[5]: http://download.microsoft.com/download/E/D/B/EDB86AD9-4D26-4C33-A8B2-82BE161682E2/WindowsBlue-KB2969050-x86.msu