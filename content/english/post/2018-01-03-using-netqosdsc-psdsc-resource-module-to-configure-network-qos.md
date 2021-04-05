---
title: 'Using NetQoSDSC #PSDSC Resource Module to Configure Network QoS'
author: Ravikanth C
type: post
date: 2018-01-03T17:00:22+00:00
url: /2018/01/03/using-netqosdsc-psdsc-resource-module-to-configure-network-qos/
views:
  - 8248
post_views_count:
  - 4238
categories:
  - PowerShell DSC
  - Module Spotlight
tags:
  - PowerShell DSC
  - Modules

---
As a part of larger hyper-converged infrastructure (based on S2D) configuration automation using PowerShell DSC, I have written quite a few new DSC resource modules. [FailoverClusterDSC][1] was one of the modules in that list. I added [NetQoSDSC][2] as well to ensure I have the automated means to configure the QoS policies in Windows Server 2016.

This module contains five resources at the moment.

### Enable/Disable Network Adapter QoS

The _NetAdapterQoS_ resource can be used to enable/disable QoS a specific network adapter.

```powershell
Configuration NetAdapterQoS
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetQoSDSC -ModuleVersion 1.0.0.0

    NetAdapterQoS EnableAdapterQoS
    {
        NetAdapterName = 'Storage1'
        Ensure = 'Present'
     }
}
```

### Enable/Disable DCBX Willing mode

DCBX willing mode can be enabled or disabled using the _NetQoSDCBXSetting_ resource. This can be done at an interface level or at the global level in the operating system.

```powershell
Configuration DisableGlobalDCBX
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetQosDsc -ModuleVersion 1.0.0.0

    NetQoSDcbxSetting DisableGlobal
    {
        InterfaceAlias = 'Global'
        Willing = $false
     }

     NetQoSDcbxSetting EnableStorage1
     {
        InterfaceAlias = 'Storage1'
        Willing = $true
      }
}
```

### Enable/Disable Network QoS flow control priorities

The _NetQosFlowControl_ resource can be used to enable or disable 802.1P flow control priorities.

```powershell
Configuration NetQoSFlowControl
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetQoSDSC -ModuleVersion 1.0.0.0

    NetQoSFlowControl EnableP3
    {
       Id = 'Priority3'
       Priority = 3
       Enabled = $true
    }

    NetQoSFlowControl DisableRest
    {
        Id = 'RestPriority'
        Priority = @(0,1,2,4,5,6,7)
        Enabled = $false
     }
}
```

### Create new QoS policies

New network QoS policies can be created using the _NetQoSPolicy_ resource.

```powershell
Configuration NewNetQoSPolicy
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetQoSDSC -ModuleVersion 1.0.0.0

    NetQosPolicy SMB
    {
        Name = 'SMB'
        PriorityValue8021Action = 3
        PolicyStore = 'localhost'
        NetDirectPortMatchCondition = 445
        Ensure = 'Present'
     }
}
```

### Manage Network QoS Traffic classes

The NetQoSTrafficClass resource can be used to manage the traffic classes in network QoS.

```powershell
Configuration NewTrafficClass
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetQoSDSC -ModuleVersion 1.0.0.0

    NetQosTrafficClass SMB
    {
        Name = 'SMB'
        Algorithm = 'ETS'
        Priority = 3
        BandwidthPercentage = 50
        Ensure = 'Present'
     }
}
```

This module, while code complete, needs some more work to declare as fully HQRM-compliant. I am working towards that by adding tests and better examples. Feel free to submit your issues, feedback, or PRs.

[1]: https://github.com/rchaganti/FailoverClusterDSC
[2]: https://github.com/rchaganti/NetQoSDSC