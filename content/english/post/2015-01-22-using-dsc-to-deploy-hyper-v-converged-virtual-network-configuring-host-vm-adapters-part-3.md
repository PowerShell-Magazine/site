---
title: Using DSC to deploy Hyper-V converged virtual network – Configuring Host VM Adapters (Part 3)
author: Ravikanth C
type: post
date: 2015-01-22T17:00:54+00:00
url: /2015/01/22/using-dsc-to-deploy-hyper-v-converged-virtual-network-configuring-host-vm-adapters-part-3/
views:
  - 10375
post_views_count:
  - 1848
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In an earlier article, I had introduced DSC resources that can be used to [deploy a Hyper-V converged virtual network][1]. In this series of articles, we will build the converged network configuration for what is shown below.

![](/images/convergednet4.png)

Here is a recap of configuration needed.

  1. [Create a host network team][1].
  2. [Create a VM switch using the host team.][2]
  3. Create host VM adapters in the management OS to connect to the VM switch and assign VLANs and bandwidth settings (today&#8217;s article).
  4. Assign IP addresses and DNS addresses, as required.

In the earlier article, we created a network team named HostTeam and deployed a VM switch using that network team. In today&#8217;s article, we will see how to create virtual network adapters that are required for the Hyper-V cluster.

### Creating VM Network adapters using cVMNetworkAdapter DSC resource

If you have not downloaded the new DSC resource modules yet, you can do so from my [Github repo][3] or using the [Install-Module cmdlet in WMF 5.0][4]. To create a converged virtual switch, we will need the [cVMNetworkAdapter][5] resource from the [cHyper-V][6] module. The cVMNetworkAdapter can be used to create VM network adapters either in the management OS or connect network adapters to VMs.

![](/images/cVMNetworkAdapter-1.png)

In this resource properties, the _Name_ represents the name of VM network adapter. The _SwitchName_ property represents the name of the virtual switch this VM network adapter should be connected to. Apart from these two mandatory properties, the _ManagementOS_ property is a mandatory property as well. If you are connecting to the VM adapter to the management OS, set this property to $true. If you are configuring the VM network adapter for a virtual machine, you must set the _ManagementOS_ property to $false and use the _VMName_ property to specify the name of the VM to which the adapter needs to connect.

By default, the VM network gets a dynamic MAC address gets assigned to it. If you need to set a static MAC address, you can use the _StaticMacAddress_ property. Here is an example configuration.

```powershell
cVMNetworkAdapter VMNetDemo {
   Name = 'NetDemo'
   SwitchName = 'HostSwitch'
   ManagementOS = $true
   Ensure = 'Present'
   DependsOn = '[cVMSwitch]HostSwitch'
}
```


Now, coming back to the converged virtual network configuration, we need to have three network adapters in the management OS &#8211; Management, Cluster, and LiveMigration. In the last article, I showed you how to create the VM switch using _cVMSwitch_ resource. Since we have set _AllowManagementOS_ to _$true_, the VM switch creation also creates a network adapter in the management OS with the same name as VM switch. So, we don&#8217;t have to create another adapter for management purpose. We can use the adapter that is created along with the VM switch for all management related traffic. That leaves us with two more adapters &#8211; Cluster and Live Migration. Here is the complete configuration that I am using to create a host team, VM switch, and finally the VM network adapters.


```powershell
Configuration DemoNetworkTeam {
   Import-DscResource -Module cWindowsOS -Name cNetworkTeam
   Import-DscResource -Module cHyper-V -Name cVMSwitch, cVMNetworkAdapter
   Node Localhost {
      cNetworkTeam NetworkTeam {
         Name = 'HostTeam'
         TeamingMode = 'SwitchIndependent'
         LoadBalancingAlgorithm = 'HyperVPort'
         TeamMembers = 'NIC1','NIC2'
         Ensure = 'Present'
      }
     cVMSwitch HostSwitch {
         Name = 'HostSwitch'
         Type = 'External'
         AllowManagementOS = $true
         MinimumBandwidthMode = 'Weight'
         NetAdapterName = 'HostTeam'
         Ensure = 'Present'
         DependsOn = '[cNetworkTeam]NetworkTeam'
      }

      cVMNetworkAdapter HostCluster {
         Name = 'HostCluster'
         SwitchName = 'HostSwitch'
         ManagementOS = $true
         Ensure = 'Present'
         DependsOn = '[cVMSwitch]HostSwitch'
      }

      cVMNetworkAdapter HostLiveMigration {
         Name = 'HostLiveMigration'
         SwitchName = 'HostSwitch'
         ManagementOS = $true
         Ensure = 'Present'
         DependsOn = '[cVMSwitch]HostSwitch'
      }
   }
}

DemoNetworkTeam
```
This will create two more network adapters in the management OS and attach them to the HostSwitch that we configured earlier.

![](/images/cVMNetworkAdapter-2.png)

### Configuring Bandwidth settings

Once the management adapters are created, we can assign the bandwidth reservation or priority settings as needed. Since we set the _MinimumBandwidthMode_ to Weight during VM switch creation, we need to specify the percentage of bandwidth reservation for each adapter.  We use [_cVMNetworkAdapterSettings_][7] DSC resource for this purpose. This DSC resource can used for many other settings such as _DhcpGuard_, _RouterGuard_ and so on.

![](/images/cVMNetworkAdapterSettings-1.png)

There are three mandatory properties similar to the _cVMNetworkAdapter_ DSC resource. You must specify the _Name_, _SwitchName_, and _ManagementOS_ properties.

The _MaximumBandwidth_ property is used to specify the maximum bandwidth, in bits per second, for the virtual network adapter. The _MinimumBandwidthAbsolute_ specifies the minimum bandwidth, in bits per second, for the virtual network adapter. By default, these properties are set to zero which means those parameters within the network adapter are disabled. The _MinimumBandwidthWeight _specifies the minimum bandwidth, in terms of relative weight, for the virtual network adapter. The weight describes how much bandwidth to provide to the virtual network adapter relative to other virtual network adapters connected to the same virtual switch.

If you want allow teaming of network adapters in the guest OS, you can set the _AllowTeaming_ property to On. By default, this is set to Off and therefore disallows network teaming inside guest OS. Similar to this, there are other settings of a VM network adapter that you can configure. These properties include _DhcpGuard_, _MacAddressSpoofing_, _PortMirroring_, _RouterGuard_, _IeeePriorityTag_, and _VmqWeight_. These properties are self explanatory and are left to defaults for a VM network adapter.

For now, we will limit our discussion only to bandwidth settings. Here is a sample configuration using this resource.

```powershell
cVMNetworkAdapterSettings HostClusterSettings {
   Name = 'HostCluster'
   SwitchName = 'HostSwitch'
   ManagementOS = $true
   MinimumBandwidthWeight = 10
   DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostCluster'
}
```


As shown in the converged network diagram at the beginning, I am giving bandwidth weight of 10, 20, and 30 to HostSwitch, HostCluster, and HostLiveMigration adapters respectively. So, for my scenario, here is the configuration script that creates a host team, VM switch, VM network adapters in the management OS, and finally configures the bandwidth settings for each adapter.


```powershell
Configuration DemoNetworkTeam {
   Import-DscResource -Module cWindowsOS -Name cNetworkTeam
   Import-DscResource -Module cHyper-V -Name cVMSwitch, cVMNetworkAdapter, cVMNetworkAdapterSettings
   Node Localhost {
      cNetworkTeam NetworkTeam {
        Name = 'HostTeam'
        TeamingMode = 'SwitchIndependent'
        LoadBalancingAlgorithm = 'HyperVPort'
        TeamMembers = 'NIC1','NIC2'
        Ensure = 'Present'
      }
      cVMSwitch HostSwitch {
        Name = 'HostSwitch'
        Type = 'External'
        AllowManagementOS = $true
        MinimumBandwidthMode = 'Weight'
        NetAdapterName = 'HostTeam'
        Ensure = 'Present'
        DependsOn = '[cNetworkTeam]NetworkTeam'
     }

     cVMNetworkAdapterSettings HostSwitchSettings {
       Name = 'HostSwitch'
       SwitchName = 'HostSwitch'
       ManagementOS = $true
       MinimumBandwidthWeight = 10
       DependsOn = '[cVMSwitch]HostSwitch'
     }

     cVMNetworkAdapter HostCluster {
       Name = 'HostCluster'
       SwitchName = 'HostSwitch'
       ManagementOS = $true
       Ensure = 'Present'
       DependsOn = '[cVMSwitch]HostSwitch'
     }

     cVMNetworkAdapterSettings HostClusterSettings {
       Name = 'HostCluster'
       SwitchName = 'HostSwitch'
       ManagementOS = $true
       MinimumBandwidthWeight = 20
       DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostCluster'
     }

     cVMNetworkAdapter HostLiveMigration {
        Name = 'HostLiveMigration'
        SwitchName = 'HostSwitch'
        ManagementOS = $true
        Ensure = 'Present'
        DependsOn = '[cVMSwitch]HostSwitch'
     }

     cVMNetworkAdapterSettings HostLiveMigrationSettings {
       Name = 'HostLiveMigration'
       SwitchName = 'HostSwitch'
       ManagementOS = $true
       MinimumBandwidthWeight = 30
       DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostLiveMigration'
    }
  }
}

DemoNetworkTeam
```
After we apply this configuration, we can verify the bandwidth settings using the _Get-VMNetworkAdapter_ cmdlet.

```powershell
Get-VMNetworkAdapter -ManagementOS | Select Name, @{Label = 'MinimumBandwidthWeight'; Expression={$_.BandwidthSetting.MinimumBandwidthWeight}} | ft -AutoSize
```


### Configuring VLAN settings

In a production environment, when using converged network configuration, you should not mix different types of traffic originating from host and virtual machines. This configuration on the network adapters can be done using the [cNetworkAdapterVlan][8] DSC resource.

![](/images/cVMNetworkAdapterVlan-1.png)

When configuring the VLAN settings for a VM network adapter, you must specify the _Name_ of the adapter and whether that belongs to _ManagementOS_ or not. If the VM adapter belongs to a VM, you should set the _ManagementOS_ property to $false and specify a VM name using the _VMName_ property. The _AdapterMode_ property specifies the operation mode of the adapter and is by default set to _Untagged_ which means there is not VLAN configuration. The possible and valid values for this property are _Untagged_, _Access_, _Trunk, Community, Isolated,_ and _Promiscuous. _Each of these modes have a corresponding VLAN property that is mandatory. For example, if you set the _AdapterMode_ property to _Access_, then it is mandatory to provide _VlanId_ property. Similarly, if you set the _AdapterMode_ to Trunk, the _NativeVlanId_ property must be specified. The following table describes the properties that must be set and the optional properties for each operating mode.

| **AdapterMode** | **Mandatory Property** | **Optional Property** |
| --------------- | ---------------------- | --------------------- |
| Untagged        | –                      | –                     |
| Access          | VlanId                 | –                     |
| Trunk           | NativeVlanId           | AllowedVlanIdList     |
| Community       | PrimaryVlanId          | SecondaryVlanId       |
| Isolated        | PrimaryVlanId          | SecondaryVlanId       |
| Promiscuous     | PrimaryVlanId          | SecondaryVlanIdList   |

Here is a sample configuration script that shows _cNetworkAdapterVlan_ resource in action.

```powershell
cVMNetworkAdapterVlan HostSwitchVlan {
   Name = 'HostSwitch'
   ManagementOS = $true
   AdapterMode = 'Access'
   VlanId = 10
   DependsOn = '[cVMSwitch]HostSwitch'
}
```


In the scenario that I am building I have to configure VLAN 10, 20, and 30 for the HostSwitch, HostCluster, and HostLiveMigration  adapters respectively. Let us see the complete configuration script now.


```powershell
Configuration DemoNetworkTeam {
    Import-DscResource -Module cWindowsOS -Name cNetworkTeam
    Import-DscResource -Module cHyper-V -Name cVMSwitch, cVMNetworkAdapter, cVMNetworkAdapterSettings, cVMNetworkAdapterVlan
    Node Localhost {
       cNetworkTeam NetworkTeam {
          Name = 'HostTeam'
          TeamingMode = 'SwitchIndependent'
          LoadBalancingAlgorithm = 'HyperVPort'
          TeamMembers = 'NIC1','NIC2'
          Ensure = 'Present'
       } 
       cVMSwitch HostSwitch {
          Name = 'HostSwitch'
          Type = 'External'
          AllowManagementOS = $true
          MinimumBandwidthMode = 'Weight'
          NetAdapterName = 'HostTeam'
          Ensure = 'Present'
          DependsOn = '[cNetworkTeam]NetworkTeam'
       }

       cVMNetworkAdapterSettings HostSwitchSettings {
          Name = 'HostSwitch'
          SwitchName = 'HostSwitch'
          ManagementOS = $true
          MinimumBandwidthWeight = 10
          DependsOn = '[cVMSwitch]HostSwitch'
       }

       cVMNetworkAdapterVlan HostSwitchVlan {
          Name = 'HostSwitch'
          ManagementOS = $true
          AdapterMode = 'Access'
          VlanId = 10
          DependsOn = '[cVMSwitch]HostSwitch'
      }

      cVMNetworkAdapter HostCluster {
          Name = 'HostCluster'
          SwitchName = 'HostSwitch'
          ManagementOS = $true
          Ensure = 'Present'
          DependsOn = '[cVMSwitch]HostSwitch'
      }

      cVMNetworkAdapterSettings HostClusterSettings {
          Name = 'HostCluster'
          SwitchName = 'HostSwitch'
          ManagementOS = $true
          MinimumBandwidthWeight = 20
          DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostCluster'
      }

      cVMNetworkAdapterVlan HostClusterVlan {
          Name = 'HostCluster'
          ManagementOS = $true
          AdapterMode = 'Access'
          VlanId = 20
          DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostCluster'
      }

      cVMNetworkAdapter HostLiveMigration {
          Name = 'HostLiveMigration'
          SwitchName = 'HostSwitch'
          ManagementOS = $true
          Ensure = 'Present'
          DependsOn = '[cVMSwitch]HostSwitch'
      }

      cVMNetworkAdapterSettings HostLiveMigrationSettings {
         Name = 'HostLiveMigration'
         SwitchName = 'HostSwitch'
         ManagementOS = $true
         MinimumBandwidthWeight = 30
         DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostLiveMigration'
     }

     cVMNetworkAdapterVlan HostLiveMigrationVlan {
         Name = 'HostLiveMigration'
         ManagementOS = $true
         AdapterMode = 'Access'
         VlanId = 30
         DependsOn = '[cVMSwitch]HostSwitch','[cVMNetworkAdapter]HostLiveMigration'
     }
   }
}

DemoNetworkTeam
```
Once we apply this configuration, it completes the creation of converged virtual network. The VLAN settings can be verified using the _Get-VMNetworkAdapterVlan_ cmdlet.

```powershell
Get-VMNetworkAdapterVlan -ManagementOS
```


This brings us to the end of today&#8217;s article. Tomorrow, we will see how we can assign IP address configuration to the VM adapters we created in this converged virtual network.

[1]: /2015/01/20/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-host-team-part-1/
[2]: /2015/01/21/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-converged-virtual-switch-part-2/
[3]: https://github.com/rchaganti/DSCResources
[4]: https://www.powershellgallery.com/
[5]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMNetworkAdapter
[6]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V
[7]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMNetworkAdapterSettings
[8]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/DSCResources/cVMNetworkAdapterVlan