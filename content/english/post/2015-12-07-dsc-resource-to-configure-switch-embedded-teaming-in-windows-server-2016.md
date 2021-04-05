---
title: PowerShell DSC Resources to Configure Switch Embedded Teaming and NAT Switch in Windows Server 2016
author: Ravikanth C
type: post
date: 2015-12-07T17:00:28+00:00
url: /2015/12/07/dsc-resource-to-configure-switch-embedded-teaming-in-windows-server-2016/
views:
  - 15310
post_views_count:
  - 2270
categories:
  - PowerShell DSC
  - Hyper-V
tags:
  - Hyper-V
  - PowerShell DSC

---
Earlier this year, I released a bunch of [DSC resources][1] packaged as [cHyper-V resource module][2]. The resources are focused on configuring Hyper-V networking. For example, you can use the resources in the cHyper-V module to configure a converged network switch on Hyper-V. These resources have been very handy for me and I built and re-built a few Hyper-V clusters with a lot of ease.

With the Server 2016 TP4 release, I wanted to take advantage of DSC as build my test labs. So, I wrote a couple of DSC resources and added to the cHyper-V resource module.

### Switch Embedded Teaming (SET)

With Windows Server 2016, there is a new networking feature called [Switch Embedded Teaming (SET)][3]. In the older method of creating a converged network, we created a native network team and then used that for VM switch. With SET, we can create a team of network adapters (up to 8) embedded within the virtual switch.

This [graphical representation of SET](https://i-technet.sec.s-msft.com/dynimg/IC822541.jpeg), from TechNet, should be helpful in understanding what needs to be achieved.

Using this DSC resource for creating Switch Embedded Teaming (SET) is straightforward. The following screenshot shows the available resource properties.

![](/images/set1.png)

Here is the sample configuration that shows how to use the resource.


```powershell
Configuration DemoSET {
    Import-DscResource -ModuleName cHyper-V -Name cSwitchEmbeddedTeam, cVMNetworkAdapter
    cSwitchEmbeddedTeam DemoSETteam {
        Name = 'MySetTeam'
        NetAdapterName = 'NIC1','NIC2'
        AllowManagementOS = $true
        Ensure = 'Present' 
    }
    
    cVMNetworkAdapter TestNet {
        Name = 'DemoAdapter'
        SwitchName = 'MySetTeam'
        ManagementOS = $true
        Ensure = 'Present'
        DependsOn = "[cSwitchEmbeddedTeam]DemoSETteam"
    }
}
```
This is all you need to do. All members that should be a part of SET can be listed as a value of the _NetAdapterName_ property. Setting _AllowManagementOS_ to _$true_ in the SET configuration creates an adapter in the management OS. The cVMNetworkAdapter is an option configuration. It is not required unless you want to add a second adapter to the management OS that is connected to the SET.

#### NAT switch Configuration

Starting with Server 2016 TP4, you can create a new VM switch type called NAT switch. This is mostly meant for container scenarios but there is nothing limiting you from taking advantage of this in a scenario that requires private connectivity within the VMs but needs external access for connecting to public Internet. This switch configuration is very simple and I wrapped up those steps in a DSC resource module.

![](/images/set2.png)

The _Name _property identifies the name of the VM switch and the _NATSubnetAddress_ properties identifies the address range that should be used for NATing.

Here is how you use this resource.

```powershell
Configuration NatDemo {
    Import-DscResource -ModuleName cHyper-V -Name cNatSwitch
    cNatSwitch NatDemo {
        Name = 'SQLNatConfig'
        NATSubnetAddress = '192.168.0.0/24'
        Ensure = 'Present'
    }
}
```


Looks like there are issues with the NAT switch configuration and NAT configuration overall. They don&#8217;t seem to be cleaning up well when we remove the configuration. So, you may find that using the same name or subnet address values for the switch creating multiple times will fail.

You can get these resource modules directly from my [Github repository][2] or from the [PowerShell Gallery][4].

Go ahead and try these new resources. Feel free to create any bug report or feature requests as needed.

[1]: https://github.com/rchaganti/DSCResources
[2]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V
[3]: https://technet.microsoft.com/en-us/library/mt403349.aspx#bkmk_sswitchembedded
[4]: https://www.powershellgallery.com/