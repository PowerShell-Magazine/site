---
title: Using DSC to deploy Hyper-V converged virtual network – Creating a host team (Part 1)
author: Ravikanth C
type: post
date: 2015-01-20T16:11:56+00:00
url: /2015/01/20/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-host-team-part-1/
views:
  - 8258
post_views_count:
  - 1204
categories:
  - PowerShell DSC
  - Hyper-V
tags:
  - PowerShell DSC
  - Hyper-V

---
In yesterday&#8217;s article, I had introduced DSC resources that can be used to [deploy a Hyper-V converged network][1]. Starting today, we will see detailed walk-through for using each of these resources we published yesterday. Using these DSC resources, we will build the converged network configuration for what is shown below.

![](/images/convergednet4.png)

Here is a recap of the configuration needed.

  1. Create a host network team (today&#8217;s article).
  2. Create a VM switch using the host team.
  3. Create host VM adapters in the management OS to connect to the VM switch and assign VLANs and bandwidth settings.
  4.  Assign IP addresses and DNS addresses, as required.

let us start!

### Creating a Host Network Team using cNetworkTeam DSC resource

If you have not downloaded the new DSC resource modules yet, you can do so from my [Github repo][2] or using the [Install-Module cmdlet in WMF 5.0][3]. To create a host network team, we will need the [cNetworkTeam][4] resource from the [cWindowsOS][5] module.

![](/images/cnet2.png)

The _cNetworkTeam_ resource has two mandatory properties. Using the _Name_ property, you can specify the name of the host team. The _TeamMembers_ property is used to specify all the network connections that should be a part of the host team.

The _LoadBalancingAlgorigthm_ is set by default to _HyperVPort_. The other possible values are _Dynamic_, _IPAddresses_, _MacAddresses_, and _TransportPorts_. The _TeamingMode_ is by default set to _SwitchIndependent_ and can be modified to either _LACP_ or _Static_.

In my scenario, to create a host team, I am using the following configuration script. Remember that you need to use the _Import-DscResource_ dynamic keyword in the configuration to load the resources from the resource module.

```powershell
Configuration HyperVConvergedNet {
   Import-DscResource -Module cWindowsOS -Name cNetworkTeam
   Node Localhost {
          cNetworkTeam NetworkTeam {
          Name = 'HostTeam'
          TeamingMode = 'SwitchIndependent'
          LoadBalancingAlgorithm = 'HyperVPort'
          TeamMembers = 'NIC1','NIC2'
          Ensure = 'Present'
      }
   }
}
```


When you apply this configuration, a host team gets created.

![](/images/cnet3.png)

In tomorrow&#8217;s article, we will see how we can create a Hyper-V VM switch that is capable of converged networking using [cVMSwitch][6] DSC resource.

[1]: /2015/01/19/announcing-dsc-resources-to-deploy-hyper-v-converged-virtual-network/
[2]: https://github.com/rchaganti/DSCResources
[3]: https://www.powershellgallery.com/
[4]: https://github.com/rchaganti/DSCResources/tree/master/cWindowsOS/DSCResources/cNetworkTeam
[5]: https://github.com/rchaganti/DSCResources/tree/master/cWindowsOS
[6]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMSwitch