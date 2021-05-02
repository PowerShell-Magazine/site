---
title: Improvements to DSC v1
author: Jan Egil Ring
type: post
date: 2015-03-09T16:00:15+00:00
url: /2015/03/09/improvements-to-dsc-v1/
views:
  - 10988
post_views_count:
  - 1287
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
KB3000850, the November 2014 update rollup for Windows RT 8.1, Windows 8.1, and Windows Server 2012 R2, has brought several enhancements and fixes to the first version of DSC (initially released as a part of Windows PowerShell 4.0 in Windows Management Framework (WMF) 4.0).

### New cmdlets and functions

Several cmdlets and functions available in the latest WMF 5.0 Preview are now available in the updated WMF 4.0 as well. In the following image the output of Get-Command -Module PSDesiredStateConfiguration from computers running WMF 4.0 both with and without KB3000850 is stored in two variables which are compared using Compare-Object to see the differences:

![](/images/WMF_V1_Update_img_02.png)

**Remove-DscConfigurationDocument** – There are configuration documents for various stages in DSC (current.mof, pending.mof, and previous.mof) stored in C:\Windows\System32\Configuration. It can happen that a need to remove one or more of these files arise, for example if a document is corrupted for some reason. It is now possible to use the new Remove-DscConfigurationDocument function to perform this task:

![](/images/WMF_V1_Update_img_01.png)

**Stop-DscConfiguration** – If a consistency check was running longer than expected, there wasn&#8217;t an easy way to stop the process before this command became available. As the name indicates, it can stop a running configuration.

**Update-DscConfiguration** – Triggers a pull request from a pull server, and thus only works for pull mode. If a new configuration is available, both the configuration and any dependent DSC Resources will be downloaded and applied. There are also several parameters available for this cmdlet worth looking up in the help system, such as –Wait and –Force.

### Updates to existing cmdlets and functions

Using Shay Levy&#8217;s script listed in [this article][1], we can quickly see what cmdlets/functions and parameters are added, removed, or changed by KB3000850:

!  > Changed

*> New

–  > Removed

New-DSCCheckSum (!)

Confirm (+)

WhatIf (+)

Start-DscConfiguration (!)

UseExisting (+)

Remove-DscConfigurationDocument (+)

Stop-DscConfiguration (+)

Update-DscConfiguration (+)

**Improvements to existing cmdlets and functions**

**New-DSCCheckSum** – Previously, there was a [bug][2] causing this function to fail when specifying an UNC path to the -OutPath parameter. This is now fixed.

**Get-DscResource** – Previously, this function was [very slow][3] to enumerate the available DSC resources. This is now fixed.

**Get-DscLocalConfigurationManager** – This function now shows more information; among the new properties shown is the LCMState which displays the current state of the DSC engine.

WMF 4.0 without KB3000850:

![](/images/WMF_V1_Update_img_03.png)

WMF 4.0 with KB3000850:

![](/images/WMF_V1_Update_img_04.png)

**Start-DscConfiguration** – Previously the –Force parameter didn't always work correctly, this is now fixed.<br />
There is also a new parameter, -UseExisting, which will apply the configuration document already present in the configuration store.<br />
Previously there wasn`t any cmdlet available to trigger a consistency check, we had to manually start the scheduled task “Consistency” or invoke the appropriate CIM method.

Now the -UseExisting parameter let us perform this task in a more convenient way.

![](/images/WMF_V1_Update_img_07.png)

**Test-DscConfiguration** – Previously, this function returned only True or False, making it hard to know what computer is referenced when running against multiple remote computers. It now returns the computer name as a value of the PSComputerName property.

WMF 4.0 without KB3000850:

![](/images/WMF_V1_Update_img_05.png)

WMF 4.0 with KB3000850:

![](/images/WMF_V1_Update_img_06.png)

[1]: /2011/09/15/how-to-find-out-whats-new-in-powershell-vnext/
[2]: https://connect.microsoft.com/PowerShell/feedback/details/812941/new-dscchecksum-fails-when-specifying-an-unc-path-to-the-outpath-parameter
[3]: https://connect.microsoft.com/PowerShell/feedback/details/812945/dsc-get-dscresource-is-very-slow