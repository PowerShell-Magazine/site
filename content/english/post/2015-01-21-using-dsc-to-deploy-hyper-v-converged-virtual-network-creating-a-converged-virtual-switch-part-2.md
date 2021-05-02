---
title: Using DSC to deploy Hyper-V converged virtual network – Creating a converged virtual switch (Part 2)
author: Ravikanth C
type: post
date: 2015-01-21T17:00:23+00:00
url: /2015/01/21/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-converged-virtual-switch-part-2/
views:
  - 9163
post_views_count:
  - 1294
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In an earlier article, I had introduced DSC resources that can be used to [deploy a Hyper-V converged virtual network][1]. In this series of articles, we will build the converged network configuration for what is shown below.

![](/images/convergednet4.png)

Here is a recap of configuration needed.

  1. [Create a host network team][1].
  2. Create a VM switch using the host team (today&#8217;s article).
  3. Create host VM adapters in the management OS to connect to the VM switch and assign VLANs and bandwidth settings.
  4.  Assign IP addresses and DNS addresses, as required.

In the earlier article, we created a network team named HostTeam. In today&#8217;s article, we will see how to create a converged virtual switch using this host team adapter.

### Creating a converged virtual switch using cVMSwitch DSC resource

If you have not downloaded the new DSC resource modules yet, you can do so from my [Github repo][2] or using the [Install-Module cmdlet in WMF 5.0][3]. To create a converged virtual switch, we will need the [cVMSwitch][4] resource from the [cHyper-V][5] module.

The _cVMSwitch_ resource is a fork from the _xVMSwitch_ resource in the [xHyper-V][6] resource module. To be able to create a converged virtual switch, we need to set the _MinimumBandwidthMode_ property. This is what the _cVMSwitch_ resource enables.

![](/images/cVMSwitch-1.png)

The _Name_ property is a mandatory one which identifies the VM switch and the _Type_ property defines the type of switch &#8211; External or Internal or Private &#8211; to be created. The _AllowManagementOS_ boolean property can be used to specify if a VM network adapter in the management OS or root partition should be created. The _MinimumBandwidthMode_ property defines how the bandwidth reservation must be done. If you specify Absolute, the subsequent VM network adapter settings can be used to specify the absolute bandwidth reservation (in mbps). By using Weight as the _MinimumBandwidthMode_, you can specify a percentage value as the bandwidth reservation for the VM network adapters. We will see more about this in the next article. Finally, the _NetAdapterName_ can be used to specify which host network adapter should be used to create an external VM switch. In our demonstration, this is the _HostTeam_ adapter created using the _cNetworkTeam_ resource.

Let us see the configuration script that creates the external converged VM switch for us.

```powershell
Configuration DemoNetworkTeam {
   Import-DscResource -Module cWindowsOS -Name cNetworkTeam
   Import-DscResource -Module cHyper-V -Name cVMSwitch

   Node Localhost {
         #Create Network Team named HostTeam and add NIC1 and NIC2 as members
         cNetworkTeam NetworkTeam {
             Name = 'HostTeam'
             TeamingMode = 'SwitchIndependent'
             LoadBalancingAlgorithm = 'HyperVPort'
             TeamMembers = 'NIC1','NIC2'
             Ensure = 'Present'
         }     
         #Create a VM Switch from HostTeam adapter and set the Bandwidth mode to weight
         cVMSwitch HostSwitch {
             Name = 'HostSwitch'
             Type = 'External'
             AllowManagementOS = $true
             MinimumBandwidthMode = 'Weight'
             NetAdapterName = 'HostTeam'
             Ensure = 'Present'
             DependsOn = '[cNetworkTeam]NetworkTeam'
         }
	}
}
```
In the above configuration, I have included both network team and the VM switch creation. I have also added the _DependsOn_ property in the _HostSwitch_ configuration to specify that the VM switch should be created only if the network team named _HostTeam_ exists. Since I set the _MinimumBandwidthMode_ to Weight, subsequent VM network adapter creation must specify the percentage of bandwidth reserve instead of absolute value.

![](/images/cVMSwitch-2.png)

This is it. For the converged virtual network, we have created the network team and the VM switch. In the later articles, we will see how to create the VM network adapters in the management OS and complete the converged virtual network configuration.

[1]: /2015/01/20/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-host-team-part-1/
[2]: https://github.com/rchaganti/DSCResources
[3]: https://www.powershellgallery.com/
[4]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMSwitch
[5]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V
[6]: https://gallery.technet.microsoft.com/scriptcenter/xHyperV-Module-PowerShell-a646ad1a