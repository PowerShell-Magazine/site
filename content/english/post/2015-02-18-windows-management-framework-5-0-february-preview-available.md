---
title: Windows Management Framework 5.0 February preview available
author: Ravikanth C
type: post
date: 2015-02-19T02:23:39+00:00
url: /2015/02/18/windows-management-framework-5-0-february-preview-available/
views:
  - 9103
post_views_count:
  - 1215
categories:
  - News
tags:
  - News

---
Windows PowerShell team announced the availability of [Windows Management Framework (WMF) 5.0 Preview February 2015][1]. This preview is [available][2] for Windows 8.1, Windows Server 2012 R2, and Windows Server 2012.

As [announced earlier][3], this WMF release continues to include both stable and experimental features. I mean a lot of them! 🙂

| **Scenario**                                                 | **Design Status** |
| ------------------------------------------------------------ | ----------------- |
| Develop DSC resources with classes in Windows PowerShell     | Stable            |
| Remove DSC documents delivered to a system                   | Stable            |
| Support for inheritance with classes in Windows PowerShell   | Experimental      |
| DSC resource script debugging                                | Experimental      |
| Support for new RefreshMode                                  | Experimental      |
| Partial configurations support mixed RefreshModes            | Experimental      |
| PSScriptAnalyzer: static code analysis of Windows PowerShell artifacts | Experimental      |
| 32-bit support for the configuration keyword in DSC          | Stable            |
| Generate Windows PowerShell cmdlets based on an OData endpoint with ODataUtils | Stable            |
| Manage .ZIP archives through new cmdlets                     | Stable            |
| Audit Windows PowerShell usage by transcription and logging  | Stable            |
| Interact with symbolic links using improved Item cmdlets     | Stable            |
| Network Switch management with Windows PowerShell            | Stable            |
| DSC authoring improvements in Windows PowerShell ISE         | Stable            |

From a DSC point of view, there are many improvements.

[Class-defined DSC resource authoring][4] is improved and there are subtle changes that are required for the old class-defined resource, if you built any.

A new _RefreshMode_ in DSC called disabled lets 3rd party configuration managers such as Chef manage a node configuration using their own agent or using the _Invoke-DscResource_ cmdlet which means we would be able to do this directly using the CIM method as well. More on this later.

Partial configurations now support mixed refresh modes including Push and Disabled. [Partial configurations are great way to enable delegated configuration authoring and management][5].

DSC resource script debugging is a great addition and my subsequent posts will talk about this.

Go ahead and start exploring WMF 5.0.

[1]: http://blogs.msdn.com/b/powershell/archive/2015/02/18/windows-management-framework-5-0-preview-february-2015-is-now-available.aspx
[2]: http://www.microsoft.com/en-us/download/details.aspx?id=30653
[3]: http://blogs.msdn.com/b/powershell/archive/2014/12/10/wmf-5-0-preview-defining-quot-experimental-designs-quot-and-quot-stable-designs-quot.aspx
[4]: /2014/10/06/class-defined-dsc-resources-in-windows-management-framework-5-0-preview/
[5]: /2014/10/02/partial-dsc-configurations-in-windows-management-framework-wmf-5-0/