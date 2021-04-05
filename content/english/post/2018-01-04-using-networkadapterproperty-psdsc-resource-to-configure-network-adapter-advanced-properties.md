---
title: 'Using NetworkAdapterProperty #PSDSC Resource to Configure Network Adapter Advanced Properties'
author: Ravikanth C
type: post
date: 2018-01-04T17:00:33+00:00
url: /2018/01/04/using-networkadapterproperty-psdsc-resource-to-configure-network-adapter-advanced-properties/
featured_image: /wp-content/uploads/2011/09/PSMagLogoWhite.jpg
ratings_users:
  - 1
ratings_score:
  - 5
ratings_average:
  - 5
views:
  - 9644
post_views_count:
  - 4729
categories:
  - Module Spotlight
  - PowerShell DSC
tags:
  - Modules
  - PowerShell DSC

---
At times we need to set the physical adapter advanced properties such as VLAN ID. This can be done using the `Set-NetAdapterAdvancedProperty` cmdlet. However, when using DSC-based configuration management, it makes sense to configure these advanced properties as well using DSC resource modules. Within the automation that I have been working on for automated deployments of Storage Spaces Direct clusters, I have the need for setting adapter advanced properties and that is what triggered me to write this new resource module &#8212; [NetworkAdapterProperty][1]. This is a part of the _NetworkingDSC_ module.

> This is not a fork of the xNetworking module. I am adding only the resources that I am developing from scratch to this NetworkingDsc. These resources will follow the HQRM guidelines.
>

This resource requires the display name of the network adapter and it can be seen by using the _Get-NetAdapterAdvancedProperty_ cmdlet. Depending on what the network adapter driver implements, this list changes between different network adapter models/vendors. Let&#8217;s see a couple of examples of using this resource.

### Configuring VLAN ID advanced property

The display name of the advanced property for VLAN configuration on physical adapter is usually _VLAN ID_.

```powershell
Configuration PhysicalAdapterVLAN
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetworkingDsc -ModuleVersion 1.0.0.0

    NetworkAdapterProperty VLAN
    {
        Id = 'S1P1VLAN'
        Name = 'SLOT 1 PORT 1'
        DisplayName =  'VLAN ID'
        DisplayValue = '102'
        Ensure = 'Present'
    }
}
```

### Configuring DCBX Mode on Mellanox CX4 adapters

This following example is specific to Mellanox CX4 adapters. These adapters support firmware or host controlled DCB exchange. We can configure this by changing the value of _DcbxMode_ property.

```powershell
Configuration PhysicalAdapterVLAN
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName NetworkingDsc -ModuleVersion 1.0.0.0

    NetworkAdapterProperty S1P1DCBX
    {
        Id = 'S1P1DCBX'
        Name = 'SLOT 1 PORT 1'
        DisplayName =  'DcbxMode'
        DisplayValue = 'Host in charge'
        Ensure = 'Present'
    }

    NetworkAdapterProperty S1P2DCBX
    {
        Id = 'S2P2DCBX'
        Name = 'SLOT 1 PORT 2'
        DisplayName =  'DcbxMode'
        DisplayValue = 'Host in charge'
        Ensure = 'Present'
    }
}
```

[1]: https://github.com/rchaganti/NetworkingDSC