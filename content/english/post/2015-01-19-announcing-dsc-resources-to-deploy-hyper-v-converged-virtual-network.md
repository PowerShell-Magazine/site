---
title: Announcing DSC resources to deploy Hyper-V converged virtual network
author: Ravikanth C
type: post
date: 2015-01-19T15:30:43+00:00
url: /2015/01/19/announcing-dsc-resources-to-deploy-hyper-v-converged-virtual-network/
views:
  - 13328
post_views_count:
  - 1619
categories:
  - PowerShell DSC
  - News
tags:
  - PowerShell DSC
  - News

---
The converged virtual network within Hyper-V deployments is a common configuration. There are multiple ways to deploy the converged network configuration. The simplest method is using [the PowerShell cmdlets][1] and a slightly complex way (if you are new to System Center) is using the [System Center Virtual Machine Manager][2]. Whatever method you choose, the idea is keep the configuration consistent across the hosts in the deployment. Now, when we talk about configuration and Windows, what is better than using [Desired State Configuration][3]?

Recently, I completed writing a bunch of custom DSC resources to deploy Hyper-V converged networks in my lab! I am happy to share that work with you all today. This release of DSC resources includes two modules and a total of four new resources and update to VM switch DSC resource written by Microsoft. [So, the number of DSC resources developed by PowerShell Magazine is now increased to 26!][4] 🙂

Before we look at DSC resources in this release, let us take a look at what a Hyper-V converged network is and then map that into the DSC resources that I built.

![](/images/hcidsc1.png)

What is shown above is a typical converged network created on a Hyper-V host. In the scenario I am demonstrating, we have multiple network ports on a physical host that are teamed together to provide load balancing as well as failover. Using the network team that we create, we deploy a Hyper-V virtual switch and add multiple network adapters in the management OS or the root partition. Also, these network adapters are given some bandwidth weight and VLAN configuration. The bandwidth configuration ensures that the host adapters do not overrun the VM network traffic or to ensure they get enough bandwidth when required.

So, to be able to complete this deployment using DSC, we need multiple resources. This is what today&#8217;s release is about.

| **Resource Name**                                            | **Module Name**                                              | **New or Updated** | **Purpose**                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| cNetworkTeam                                                 | [cWindowsOS](https://github.com/rchaganti/DSCResources/tree/master/cWindowsOS) | New                | Create a Network Team from physical network interfaces.      |
| [cVMSwitch](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMSwitch) | [cHyper-V](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V) | Updated            | Create Hyper-V Virtual Switch that can be used for converged networking. **Note:** xVMSwitch from [xHyper-V](https://gallery.technet.microsoft.com/scriptcenter/xHyperV-Module-PowerShell-a646ad1a) was the base for this resource. I added changes to configure Bandwidth reservation mode and IoV settings. |
| [cVMNetworkAdapter](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMNetworkAdapter) | [cHyper-V](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V) | New                | Create and add a new VM network adapter to either the management OS or a Hyper-V virtual machine. |
| [cVMNetworkAdapterSettings](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMNetworkAdapterSettings) | [cHyper-V](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V) | New                | Configure bandwidth reservation and other settings on the VM adapter in both management OS or VM. |
| [cVMNetworkAdapterVlan](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMNetworkAdapterVlan) | [cHyper-V](https://github.com/rchaganti/DSCResources/tree/master/cHyper-V) | New                | Configure VLAN settings on the VM adapter in both management OS or VM. |

While my application of the following DSC resources is deploying a Hyper-V converged network, their functionality is more generic and granular. I could have combined _cVMNetworkAdapter_, _cVMNetworkAdapterSettings_, and _cVMNetworkAdapterVlan_ into a single resource. However, that would only increase the complexity of authoring and have too many resource properties to deal with.

**Note:** The xHyper-V resource module contains additional resources for dealing with VMs and VHDs. I have not added them except the _cVMSwitch_ to the cHyper-V resource module as I have not modified any of them.

You can download these resource modules from my [Github repo][4]. These resource modules are available on [PowerShell Gallery][5] as well. So, if you are running WMF 5.0, you can download these using the _Install-Module_ cmdlet.

```powershell
Install-Module -Name cWindowsOS
Install-Module -Name cHyper-V
```


In today&#8217;s article, I have only introduced these resources and their purpose. I will walk-through each of these resources and show you how to build a Hyper-V converged network. Stay tuned!

Meanwhile, feel free to explore the resources yourself (no rocket science, really!) and let me know what you think.

[1]: http://en.community.dell.com/techcenter/virtualization/w/wiki/4295.hyper-v-3-0-and-converged-networks-on-12th-generation-dell-poweredge-servers/
[2]: http://charbelnemnom.com/2014/05/create-a-converged-network-fabric-in-vmm-2012-r2/
[3]: /tag/dsc/
[4]: https://github.com/rchaganti/DSCResources
[5]: http://www.powershellgallery.com