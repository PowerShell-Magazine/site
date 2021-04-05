---
title: '#PSDSC Configuration Versioning in a Deployment Pipeline'
author: Ravikanth C
type: post
date: 2018-01-05T17:00:00+00:00
url: /2018/01/05/psdsc-configuration-versioning-in-a-deployment-pipeline/
views:
  - 11729
post_views_count:
  - 5273
categories:
  - PowerShell DSC
  - DevOps
tags:
  - DevOps
  - PowerShell DSC

---
When we talk about applications or software deployed in the infrastructure, we simply **refer to the version of the application** **or software** running in the infrastructure. At any point in time, we can look at the installed service or software and understand what version of that is currently running on the system, we need to remember that [selling is service][1] not sales . How about your node configurations? Does your node tell you what version of the configuration it is currently using?

For example, consider that you have a set of web server nodes and each configured using PowerShell DSC. As a part of the initial configuration, you deployed a set of web applications on the node. And, at some point in time later, you made multiple changes to the node configuration in terms of adding or removing web applications and updating application configurations. If multiple such source controlled configurations are deployed on each of these nodes, how exactly do you figure out what version of the node configuration is being used on each of the nodes? This can probably be achieved by a complete suite of tools but is there a way, today, you can do this by querying the node itself? The answer is no. At least, in the PowerShell DSC world.

One of the core concepts of Infrastructure as Code (IaC) is to version/source control your infrastructure configurations so that it becomes easy to track what version of the configuration a target node has and rollback to last known working configuration when needed. But, the compiled DSC MOF files do not carry this information today nor they are aware of any source / version control systems. They need not be aware as well.

At this point in time, I track my node configuration version by adding a DSC resource that caches the node configuration version information. As a part of the build process, I can update the version of the configuration using this DSC resource. When I need to know what version of configuration a node is using, I simply invoke the _Get-DscConfiguration_ cmdlets. to verify that.

I packaged this DSC resource into a module of its own and published in my Github account.

### ConfigurationDSC resource module

Through the [ConfigurationDSC][2] resource module, I plan to combine various resources that help in managing DSC based configurations in deployment pipeline. At this point in time, there is only one resource, [ConfigurationVersion][3], which helps in tracking the version of the configuration document. Here is an example of a configuration document with the _ConfigurationVersion_ DSC resource.

```powershell
Configuration VersionedConfiguration
{
    param
    (
        [Parameter(Mandatory = $true)]
        [String] $ConfigurationName,

        [Parameter(Mandatory = $true)]
        [String] $ConfigurationVersion
    )

    Import-DscResource -ModuleName PSDesiredStateConfiguration -ModuleVersion 1.1
    Import-DscResource -ModuleName ConfigurationDsc -ModuleVersion 1.0.0.0

    ConfigurationVersion WebServerConfiguration
    {
        Name = $ConfigurationName
        Version = $ConfigurationVersion
    }

    WindowsFeature WebServer
    {
        Name = 'Web-Server'
        IncludeAllSubFeature = $true
        Ensure = 'Present'
    }
}

VersionedConfiguration -ConfigurationName 'WebServerConfig' -ConfigurationVersion '1.0.0.0'
```

Once this configuration is enacted, I can use the `Get-DscConfiguration` cmdlet to find the version of the node configuration.

```powershell
Get-DscConfiguration | Where-Object { $_.CimClassName -eq 'ConfigurationVersion' } | Select -ExpandProperty Version
```

What I have shown here is only a workaround. This approach has both pros and cons. I rather want this added into the existing DSC feature set. If you see a compiled MOF, it has a few configuration meta properties. For example, from the above example, the compiled MOF has the following instance of the `MSFT_ConfigurationDocument` class in the MOF.

```powershell
instance of OMI_ConfigurationDocument
{
    Version="2.0.0";
    MinimumCompatibleVersion = "1.0.0";
    CompatibleVersionAdditionalProperties= {"Omi_BaseResource:ConfigurationName"};
    Author="Administrator";
    GenerationDate="01/05/2018 11:07:56";
    GenerationHost="MGMT01";
    Name="VersionedConfiguration";
};
```


It would be ideal for us to be able to add the configuration version from the source / version control system to this instance of `OMI_ConfigurationDocument`. I created an user voice item for this: <https://windowsserver.uservoice.com/forums/301869-powershell/suggestions/32825272-enable-configuration-version-tracking-for-compiled>

Go ahead and it vote it up if you think this will be useful in your infrastructure as well.

[1]: https://www.salesforce.com/blog/2013/07/selling-service-not-sales.html
[2]: https://github.com/rchaganti/ConfigurationDsc
[3]: https://github.com/rchaganti/ConfigurationDsc/tree/master/DscResources/ConfigurationVersion