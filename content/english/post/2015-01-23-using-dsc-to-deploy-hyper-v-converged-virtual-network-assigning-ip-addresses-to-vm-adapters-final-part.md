---
title: Using DSC to deploy Hyper-V converged virtual network – Assigning IP addresses to VM Adapters (Final Part)
author: Ravikanth C
type: post
date: 2015-01-23T17:00:06+00:00
url: /2015/01/23/using-dsc-to-deploy-hyper-v-converged-virtual-network-assigning-ip-addresses-to-vm-adapters-final-part/
views:
  - 9992
post_views_count:
  - 1527
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
  3. [Create host VM adapters in the management OS to connect to the VM switch and assign VLANs and bandwidth settings][3].
  4. Assign IP addresses and DNS addresses, as required (today&#8217;s article)

In the earlier articles, we created a network team named HostTeam, deployed a VM switch using that network team, and finally created a few VM network adapters and configured bandwidth and VLAN settings on them. In today&#8217;s article, we will see how to assign IP addresses to these virtual network adapters.

To complete this task, we need the _xIPAddress_ and _xDnsServerAddress_ DSC resources from the [_xNetworking_][4] module in the DSC resource kit.

In the scenario that we are working on, we need static IP addresses assigned to each of the adapters in the management OS. Also, the host management adapter requires DNS server address so that we can use that network connection to Active Directory and other infrastructure services.

Here is the configuration that I pushed to the Hyper-V server to configure IP addresses for each of the network adapters in the management OS.

```powershell
xIPAddress MgmtAddress
{
    IPAddress = "100.194.14.114"
    InterfaceAlias = "vEthernet (HostSwitch)"
    SubnetMask = 23
    DefaultGateway = '100.194.14.1'
    AddressFamily = "IPV4"
}

xDNSServerAddress MgmtDNS {
    InterfaceAlias = 'vEthernet (HostSwitch)'
    Address = '100.194.14.87'
    AddressFamily = 'IPV4'
}

xIPAddress ClusterAddress
{
    IPAddress = "172.168.1.114"
    InterfaceAlias = "vEthernet (HostCluster)"
    SubnetMask = 16
    AddressFamily = "IPV4"
}

xIPAddress LMAddress
{
    IPAddress = "192.168.1.114"
    InterfaceAlias = "vEthernet (HostLiveMigration)"
    SubnetMask = 24
    AddressFamily = "IPV4"
}
```

This completes the configuration of IP addresses and DNS addresses for the network adapters. The host management adapter needs the gateway and DNS server addresses. We don&#8217;t have to configure those settings for the cluster and live migration networks.

Make note of the value provided for the _InterfaceAlias_ property. This is not just the network adapter name we used so far but the name of the adapter as you see in network connections. For example, vEthernet (HostSwitch) should be the interface alias for the HostSwitch adapter.

![](/images/cIPAddress.png)

The following snippet provides the complete resource configuration for creating Hyper-V converged virtual network.

```powershell
Configuration DemoNetworkTeam {
    Import-DscResource -Module cWindowsOS -Name cNetworkTeam
    Import-DscResource -Module cHyper-V -Name cVMSwitch, cVMNetworkAdapter, cVMNetworkAdapterSettings, cVMNetworkAdapterVlan
    Import-DscResource -Module xNetworking -Name MSFT_xIPAddress, MSFT_xDNSServerAddress

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

xIPAddress MgmtAddress
{
    IPAddress = "100.194.14.114"
    InterfaceAlias = "vEthernet (HostSwitch)"
    SubnetMask = 23
    DefaultGateway = '100.194.14.1'
    AddressFamily = "IPV4"
}

xDNSServerAddress MgmtDNS {
    InterfaceAlias = 'vEthernet (HostSwitch)'
    Address = '100.194.14.87'
    AddressFamily = 'IPV4'
}

xIPAddress ClusterAddress
{
   IPAddress = "172.168.1.114"
   InterfaceAlias = "vEthernet (HostCluster)"
   SubnetMask = 16
   AddressFamily = "IPV4"
}

xIPAddress LMAddress
{
    IPAddress = "192.168.1.114"
    InterfaceAlias = "vEthernet (HostLiveMigration)"
    SubnetMask = 24
    AddressFamily = "IPV4"
}

}

}

DemoNetworkTeam
```

This brings us to the end of this series of articles on using DSC for creating Hyper-V converged virtual network. If you have any feedback or see any issues with the DSC resource modules, feel free to create a discussion or issue on Github. I will be more than happy to help you and make these resource modules better.

[1]: /2015/01/20/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-host-team-part-1/
[2]: /2015/01/21/using-dsc-to-deploy-hyper-v-converged-virtual-network-creating-a-converged-virtual-switch-part-2/
[3]: /2015/01/22/using-dsc-to-deploy-hyper-v-converged-virtual-network-configuring-host-vm-adapters-part-3/
[4]: https://gallery.technet.microsoft.com/scriptcenter/xNetworking-Module-818b3583