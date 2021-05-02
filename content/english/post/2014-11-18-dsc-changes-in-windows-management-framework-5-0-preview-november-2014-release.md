---
title: DSC changes in Windows Management Framework 5.0 Preview November 2014 release
author: Ravikanth C
type: post
date: 2014-11-19T03:58:53+00:00
url: /2014/11/18/dsc-changes-in-windows-management-framework-5-0-preview-november-2014-release/
categories:
  - PowerShell DSC
  - News
tags:
  - PowerShell DSC
  - News
---
If you&#8217;ve missed Windows PowerShell team&#8217;s announcement, [WMF 5.0 Preview November 2014 release is out][1].

This Windows Management Framework (WMF) preview includes everything from WMF 5.0 Preview September 2014 plus some new improvements. You’ll find improvements in:

  * OneGet
  * PowerShellGet
  * Windows PowerShell Desired State Configuration
  * Classes for Windows PowerShell
  * Debugging in Windows PowerShell

### Changes to Desired State Configuration

In addition to the major new features in last WMF 5.0 preview release such as [Partial Configurations][2], [class-defined DSC resources][3], new LCM configuration method, and several new DSC cmdlets, DSC in WMF 5.0 Preview November 2014 release includes the following enhancements.

#### 32-bit support

DSC configurations can now be defined and compiled within a 32-bit process. This means you can use a 32-bit PowerShell host to write configuration on a x64 computer.

#### Reporting Configuration Status to Central Location

You can configuration Local Configuration Manager (LCM) to send the status of the configuration to a central location. This applies to both Push and Pull clients.

The _GetErrorReport_ method of the DSC service OData endpoint provides the status of all target systems.

#### Start-DscConfiguration supports verbose output after target system restart

If a configuration requires a computer restart, _Start-DscConfiguration_ continues streaming information after the target node restarts.

[1]: http://blogs.msdn.com/b/powershell/archive/2014/11/18/windows-management-framework-5-0-preview-november-2014-is-now-available.aspx
[2]: /2014/10/02/partial-dsc-configurations-in-windows-management-framework-wmf-5-0/
[3]: /2014/10/06/class-defined-dsc-resources-in-windows-management-framework-5-0-preview/