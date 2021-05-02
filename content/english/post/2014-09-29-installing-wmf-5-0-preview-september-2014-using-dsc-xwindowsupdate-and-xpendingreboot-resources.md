---
title: Installing WMF 5.0 Preview September 2014 using DSC xWindowsUpdate resource
author: Ravikanth C
type: post
date: 2014-09-29T16:13:27+00:00
url: /2014/09/29/installing-wmf-5-0-preview-september-2014-using-dsc-xwindowsupdate-and-xpendingreboot-resources/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
If you have missed the announcement, [Powershell team released wave 7 of the DSC Resource Kit][1] just before the weekend.

This wave has added the following:

  * [xPendingReboot][2] examines three specific registry locations where a Windows Server might indicate that a reboot is pending and allows DSC to predictably handle the condition
  * [xCredSSP][3] enables or disables the server and client CredSSP roles on a system.
  * [xAdcsCertificationAuthority][4] this resource installs and configures the Certificate Authority role on a Windows Server.
  * [xAdcsWebEnrollment][4] this resource configures Certificate Services Web Enrollment on a Windows Server following installation of the component using the WindowsFeature resource.

Out of the four new DSC resources, the xPendingReboot resource is of immediate use for me. I build VMs for my lab quite often and having the newest DSC bits is a must for me. And, what is the better way than using DSC for installing WMF? 🙂

If you have installed WMF bits in the past, you will know that it requires a reboot at the end of installation procedure. xHotfix resource in the xWindowsUpdate module triggers a reboot if a Windows update requires doing so. <del>This is where xPendingReboot is useful.</del> If Local Configuration Manager (LCM) is already configured to reboot the node as needed ([_RebootNodeIfNeeded_ is set to _True_][5]), this resource can be used to induce the restart after WMF install. This resource does that by changing the _$Global:DSCMachineStatus_ variable to 1.

So, here is the configuration script I created for the purpose of installing WMF 5.0 bits on my lab systems. This requires <del>xPendingReboot and</del> xWindowsUpdate resource from the DSC Resource Kit.

**Update:** xHotfix can reboot a system if a Windows Update package install requires reboot. Therefore, using xPendingReboot is not necessary.

<del><strong>Note:</strong> DSC Resource Kit wave 7 release excluded the <a href="http://gallery.technet.microsoft.com/xWindowsUpdate-Module-with-5af00a7f">xWindowsUpdate</a> resource for some reason. If you don&#8217;t have it on your system, you can download it from <a href="http://gallery.technet.microsoft.com/xWindowsUpdate-Module-with-5af00a7f">http://gallery.technet.microsoft.com/xWindowsUpdate-Module-with-5af00a7f</a>. Also, note that these resources must be present on the target systems where you are installing WMF 5.0 using DSC.</del>

**Update**: xWindowsUpdate resource is now added to the wave 7 resource kit download.

Once you have the DSC resources copied to the VM, you can use the following configuration script that I created for installing WMF 5.0 using xWindowsUpdate <del>and xPendingReboot</del> resource<del>s</del>.

```
Configuration WMF5Sep14Install
{
   Import-DscResource -ModuleName xWindowsUpdate, xPendingReboot
   Node localhost
   {
      xHotfix HotfixInstall
      {
          Path = 'C:\Hotfix\KB2969050-x64.msu'
          Id = 'KB2969050'
          Ensure = 'Present'
      }
   }
}

WMF5Sep14Install
```

In the above example, I have downloaded the MSU package and stored it in C:\Hotfix folder. So, I provided that path as the value of _Path_ property in the xHotfix resource configuration. When we enact this configuration using the _Start-DscConfiguration_ cmdlet, we will see that the MSU gets installed and then a reboot gets induced if you have set the LCM to reboot as needed or a message will be displayed that a reboot is pending.

WMF 5.0 in this article is just an example. You can use this method to perform any number of MSU installs and actually perform the reboot as required.

[1]: http://blogs.msdn.com/b/powershell/archive/2014/09/26/continuing-the-dsc-resource-kit-additions-wave-7-is-live.aspx
[2]: http://gallery.technet.microsoft.com/xPendingReboot-PowerShell-b269f154
[3]: http://gallery.technet.microsoft.com/xCredSSP-PowerShell-Module-81414139
[4]: http://gallery.technet.microsoft.com/xAdcsDeployment-PowerShell-cc0622fa
[5]: http://technet.microsoft.com/en-us/library/dn249922.aspx