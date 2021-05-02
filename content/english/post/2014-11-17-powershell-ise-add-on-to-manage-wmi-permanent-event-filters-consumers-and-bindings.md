---
title: PowerShell ISE add-on to manage WMI permanent event filters, consumers, and bindings
author: Ravikanth C
type: post
date: 2014-11-17T17:00:21+00:00
url: /2014/11/17/powershell-ise-add-on-to-manage-wmi-permanent-event-filters-consumers-and-bindings/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
When I am working with Windows PowerShell, PowerShell ISE is my second home. I prefer to do everything within PowerShell ISE and not move away from it to other tools. If you have followed my recent posts, I&#8217;ve released a custom DSC module for managing WMI permanent event filters, consumers, and bindings. While experimenting with this module, I had created multiple instances of these WMI objects. There were different ways to delete these objects. [Trevor had created a WMIEventHelper utility][1] and [Boe converted that it into a WPF UI][2]. I could, of course, delete these objects using PowerShell itself. However, none of this was really integrated into the PowerShell ISE UI.

This is what today&#8217;s post is about. I&#8217;ve created an [PowerShell ISE add-on][3] for deleting the filters, consumers, and bindings I&#8217;d created using my [DSC resource module][4].

![](/images/events.png)

The [source code for this PowerShell ISE add-on][5] is available on Github and you can compile it using Visual Studio ([Community Edition][6]?) 🙂

Once you compile the DLL, you can load it using the Add-Type cmdlet and then add it to the vertical add-ons in ISE.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -Path 'C:\Script\PSMag.dll'
$psISE.CurrentPowerShellTab.VerticalAddOnTools.Add('Events',[PSMag.WMIEventsAddon],$true)
</pre>

I have only included the delete functionality in this initial release. While it is not tough to provide create functionality, it is low on my priority list at this moment. Since I will be using only the DSC resource module to create these WMI objects, I am exploring the ability to generate a DSC configuration script for each of the instances instead of using WMI directly.

The next update will have exploring instance details by double-clicking any of the instances. Stay tuned.

[1]: http://trevorsullivan.net/2011/01/18/removing-permanent-wmi-event-registrations/
[2]: http://learn-powershell.net/2013/08/16/introducing-posheventui-a-ui-way-to-create-permanent-wmi-events/
[3]: https://github.com/rchaganti/ISEAddons/tree/master/PSMag
[4]: https://github.com/rchaganti/DSCResources/tree/master/DSCResources
[5]: https://github.com/rchaganti/ISEAddons
[6]: http://www.visualstudio.com/products/visual-studio-community-vs