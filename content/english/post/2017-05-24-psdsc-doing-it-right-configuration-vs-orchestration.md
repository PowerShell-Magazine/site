---
title: '#PSDSC Doing It Right – Configuration vs Orchestration'
author: Ravikanth C
type: post
date: 2017-05-24T16:00:49+00:00
url: /2017/05/24/psdsc-doing-it-right-configuration-vs-orchestration/
views:
  - 12479
post_views_count:
  - 4686
categories:
  - DevOps
  - PowerShell DSC
tags:
  - DevOps
  - PowerShell DSC
---
In the last part of this series, we looked at why [resource granularity](/2017/05/23/psdsc-doing-it-right-resource-granularity/) is important. In this part, we will see the difference between configuration items and orchestration steps.

When using any configuration management platform or a tool, the imperative resource modules, that &#8220;make it so&#8221;, are the most important. The declarative configuration documents combine the resource definitions and the imperative scripts get called behind the scenes to do the job. Now, it can be very tempting to write a resource for everything going by the resource granularity principle. But, this is where we need to apply some filters. There are many such filters but let&#8217;s first start with the configuration vs orchestration filter which is the subject of this article.

Take a look at this flowchart for an example.

![](/images/dscorch1.png)

The flowchart above is about deploying OS through automated processes in my lab. This is very high-level and abstracted. Notice the step before complete OS deploy. We need to set the bare metal system to perform one time PXE boot so that the WDS server kick starts the WinPE boot and completes required OS deployment steps. In a scenario where you have artifacts developed for even bare metal configuration, it is certainly possible to put this one time PXE boot also as a resource configuration. After all, it is just BIOS attribute configuration.

I have put together an example for this.

```powershell
$configData = 
@{
    AllNodes = 
    @(
        @{
            NodeName = 'localhost'
            PSDscAllowPlainTextPassword = $true
        }
    )
}

Configuration PEPXEBootDemo
{
    Import-DscResource -ModuleName DellPEWsManTools -Name PEOneTimeBoot
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node $AllNodes.NodeName {
        PEOneTimeBoot PEPXEBootDemo
        {
            Id = 'Node17-UniqueBoot'
            DRACIPAddress = '172.16.100.17'
            DRACCredential = (Get-Credential)
            OneTimeBootMode = 'Enabled'
            OneTimeBootDevice = 'NIC.PxeDevice.1-1'
        }
    }
}

PEPXEBootDemo -ConfigurationData $configData
```

The above configuration document uses the _PEOneTimeBoot_ resource from the _DellPEWSManTools_ resource module. The _PEOneTimeBoot_ is a proxy DSC resource.

Here is what it does when we enact this configuration.

![](/images/dscorch2.png)

All is well and good. Once the bare metal system restarts, it boots into PXE and completes OS deployment. Now, here is the interesting part. If I use the _Test-DscConfiguration_ cmdlet to check if my node is in desired state or not, here is what I see.

![](/images/dscorch3.png)

There is a configuration drift. And, it will always be like this. This is because the _PEOneTimeBoot_ resource is configuring a BIOS attribute that is short lived. It gets reset or gets disabled after the configuration is complete. So, when we check the attribute after the configuration enact is complete, the _Test-DscConfiguration_ will always find that this resource has drifted from the expected configuration. When we aggregate the resource desired state at the complete system level, this drift in PEOneTimeBoot will roll up as drift at the whole system level. And, this is what makes this, _PEOneTimeBoot,_ an incorrect choice for creating a DSC resource.

In practice, this is an orchestration step and not a configuration item. As a part of the deployment process orchestration, we would need an orchestration step that PXE boots the bare metal system for OS deployment to complete and then perform any other post-OS configuration tasks using DSC.

Therefore when designing and developing a DSC resource module, apply this filter to check if the resource configuration in question is short lived. In other terms, check if the resource configuration implies to be an orchestration step or a configuration item.

Stay tuned for more in this series!

[1]: http://www.powershellmagazine.com/tag/psdscdir/
[2]: /2017/05/23/psdsc-doing-it-right-resource-granularity/