---
title: 'Survey: PowerShell DSC Composite Resources'
author: Ravikanth C
type: post
date: 2016-04-22T13:41:28+00:00
url: /2016/04/22/survey-powershell-dsc-composite-resources/
views:
  - 11941
post_views_count:
  - 2914
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
While at the PowerShell Conference EU, we had some very good discussions around PowerShell and PowerShell DSC. A few questions came up about DSC composite resources and how they are being used. DSC composite resources are essentially configurations packaged as resource modules. The benefit is that you can discover them like other DSC resource modules and use them like that. There is a good [overview and information on authoring Composite resources][1] on MSDN. So, if you are completely new to this concept, I suggest you read the article first before proceeding.

**TL;DR:** If you already know about what DSC composite resources are and how they work, We would like to understand a few aspects around composite resources. Here is a small survey for you: <https://www.surveymonkey.com/r/6K6FGTR>.

In the following example, I am using the _ServiceSet_ composite resource that comes with Windows Server 2016 TP4.

![](/images/sur1.png)

Instead of specifying multiple instances of Service resource, we can use the _ServiceSet_ to configure multiple services that need to be configured in a similar way. Here is an example of the configuration that uses _ServiceSet_ composite resource.

```powershell
Configuration DemoComposite {
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    ServiceSet ServiceSetComposite {
        Name = 'audiosrv','winmgmt'
        Ensure = 'Present'
    }
}

DemoComposite -OutputPath C:\DemoComposite

Start-DscConfiguration -Path C:\DemoComposite -Force -Wait -Verbose
```

Once you enact this configuration and use the Get-DscConfiguration cmdlet, you will see that the _ServiceSet_ resource gets expanded into the two _Service_ resource instances defined in the configuration.

```powershell
PS C:\> Get-DscConfiguration

ConfigurationName    : DemoComposite
DependsOn            : 
ModuleName           : PSDesiredStateConfiguration
ModuleVersion        : 1.1
PsDscRunAsCredential : 
ResourceId           : [Service]Resource0::[ServiceSet]ServiceSetComposite
SourceInfo           : 
BuiltInAccount       : LocalService
Credential           : 
Dependencies         : {AudioEndpointBuilder, RpcSs}
Description          : Manages audio for Windows-based programs.  If this service is stopped, audio devices and effects will not function properly.  If this service is 
                       disabled, any services that explicitly depend on it will fail to start
DisplayName          : Windows Audio
Ensure               : 
Name                 : audiosrv
Path                 : C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
StartupType          : Manual
State                : Running
Status               : 
PSComputerName       : 
CimClassName         : MSFT_ServiceResource

ConfigurationName    : DemoComposite
DependsOn            : 
ModuleName           : PSDesiredStateConfiguration
ModuleVersion        : 1.1
PsDscRunAsCredential : 
ResourceId           : [Service]Resource1::[ServiceSet]ServiceSetComposite
SourceInfo           : 
BuiltInAccount       : LocalSystem
Credential           : 
Dependencies         : {RPCSS}
Description          : Provides a common interface and object model to access management information about operating system, devices, applications and services. If this 
                       service is stopped, most Windows-based software will not function properly. If this service is disabled, any services that explicitly depend on it will 
                       fail to start.
DisplayName          : Windows Management Instrumentation
Ensure               : 
Name                 : winmgmt
Path                 : C:\Windows\system32\svchost.exe -k netsvcs
StartupType          : Automatic
State                : Running
Status               : 
PSComputerName       : 
CimClassName         : MSFT_ServiceResource
```

Now, for some people, this may not make sense. In the configuration script, we have only one resource but in the _Get-DscConfiguration_ output, we have two service instances. This can be confusing at times. The same applies to the _Test-DscConfiguration_ cmdlet as well.

```powershell
PS C:\> Test-DscConfiguration -Detailed | Select -ExpandProperty ResourcesInDesiredState

ConfigurationName    : DemoComposite
DependsOn            : 
ModuleName           : PSDesiredStateConfiguration
ModuleVersion        : 1.1
PsDscRunAsCredential : 
ResourceId           : [Service]Resource0::[ServiceSet]ServiceSetComposite
SourceInfo           : ::2::1::Service
DurationInSeconds    : 0
Error                : 
FinalState           : 
InDesiredState       : True
InitialState         : 
InstanceName         : Resource0::[ServiceSet]ServiceSetComposite
RebootRequested      : False
ResourceName         : Service
StartDate            : 4/22/2016 6:50:41 PM
PSComputerName       : localhost

ConfigurationName    : DemoComposite
DependsOn            : 
ModuleName           : PSDesiredStateConfiguration
ModuleVersion        : 1.1
PsDscRunAsCredential : 
ResourceId           : [Service]Resource1::[ServiceSet]ServiceSetComposite
SourceInfo           : ::8::1::Service
DurationInSeconds    : 0.015
Error                : 
FinalState           : 
InDesiredState       : True
InitialState         : 
InstanceName         : Resource1::[ServiceSet]ServiceSetComposite
RebootRequested      : False
ResourceName         : Service
StartDate            : 4/22/2016 6:50:41 PM
PSComputerName       : localhost
```

You can identify the resource instances that belong to the composite resource configuration using the _InstanceName_ property. However, it is not always desired.

Instead of this expansion, it will be helpful if the _Get_ and _Test_ methods roll-up the status of the resource instances from the composite resource configuration into a single instance.

So, here is a [survey for you][2]. Vote it up and let the team know what your preference is.

[1]: https://msdn.microsoft.com/en-us/powershell/dsc/authoringresourcecomposite
[2]: https://www.surveymonkey.com/r/6K6FGTR