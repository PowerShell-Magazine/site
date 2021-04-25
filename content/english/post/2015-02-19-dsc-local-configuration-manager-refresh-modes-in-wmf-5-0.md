---
title: DSC Local Configuration Manager Refresh Modes in WMF 5.0
author: Ravikanth C
type: post
date: 2015-02-19T15:00:47+00:00
url: /2015/02/19/dsc-local-configuration-manager-refresh-modes-in-wmf-5-0/
views:
  - 15046
post_views_count:
  - 2766
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In the WMF 5.0 Preview February release, a new DSC RefreshMode setting is added to the LCM meta-configuration. If you have worked with first release of DSC or any of the earlier WMF 5.0 Previews, you will know that there were two refresh modes possible -- Push and Pull. These are the configuration delivery modes and define how the configuration gets delivered to the target node. Configuration delivery essentially means how the configuration MOF document is sent to the target node and what happens thereafter. Once a configuration MOF document is available on the target node as pending.mof, it gets enacted and gets stored as current.mof, if everything goes well.

**Push** is the default configuration delivery mode and an administrator can push and enact target node configuration using the Start-DscConfiguration cmdlet. When you have hundreds if not thousands of target nodes, this method isn't scalable. Also, when using Push mode, there is no way we can automatically deliver the missing resource modules to the target nodes.

**Pull** mode enables a more centralized way of delivery configuration and resource modules to target nodes. You can either deploy the DSC service feature and configure a REST-based endpoint for pull mode or use a simple SMB file share to stage configuration documents and resource modules. In WMF 5.0, it is possible for a target node to get a subset of configuration from a REST-based pull service and the remaining from a SMB file share.

With February preview release, you can also mix Push and Pull mode of configuration delivery on the same target node when using the [partial configurations feature][1]. More on this later.

There is an additional RefreshMode setting available starting with the February preview. This setting is called 'Disabled'. What it does? As you may have correctly guessed, it disables any kind of document processing. We can use the Set-DscLocalConfigurationManager to set RefreshMode to Disabled.

```powershell
[DscLocalConfigurationManager()]
Configuration Meta {
    Node WMF5-2 {
       Settings {
           RefreshMode = "Disabled"
       }
    }
}

Meta

$CimSession = New-CimSession -ComputerName WMF5-2
Set-DscLocalConfigurationManager -Path .\Meta -CimSession $CimSession
Get-DscLocalConfigurationManager -CimSession $CimSession
Remove-CimSession -CimSession $CimSession
```

This will disable any configuration document processing on the target node.

![](/images/1.png)

When the RefreshMode is set to Disabled, you can't push the configuration using the Start-DscConfiguration cmdlet which makes perfect sense.

```powershell
Configuration DemoRefreshMode {
    Node WMF5-2 {
       File DemoFile {
          DestinationPath = 'C:\DemoRefreshMode'
          Type = 'Directory'
          Ensure = 'Present'
       }
   }
}
DemoRefreshMode

Start-DscConfiguration -ComputerName WMF5-2 -Path .\DemoRefreshMode -Verbose -Wait
```

![](/images/2.png)

You cannot force the push mode using _-Force_ switch parameter on the Start-DscConfiguration cmdlet. So, why do we need something like this?

Remember, DSC is not just a configuration management toolset. It is a configuration management platform. The already popular configuration management tools such as Chef, Puppet, and others are expected to leverage DSC for configuration management on Windows OS. When you have a 3rd party equivalent of LCM sitting on the same system managing configuration based on its own policies, you don&#8217;t want LCM to come in its way and start doing consistency checks on a periodic basis. That does not make sense. Disabling any document processing by setting LCM _RefreshMode_ is the right way here. If we cannot use Push or Pull methods, how the 3rd party configuration managers are supposed to use DSC to enact resource configuration?

For this purpose, a new cmdlet called Invoke-DscResource is introduced in the February preview. This cmdlet is supported only when the RefreshMode is set to Disabled.

![](/images/3.png)

This cmdlet takes three mandatory parameters.

**Name** of the DSC resource.

**Method** within the DSC resource that needs to be invoked.

**Property** hash table that needs to be passed to the resource method being called.

The **Module** parameter is optional and can be used to specify the name of the module for the DSC resource.

How DSC uses a resource methods? It first runs the _Test-TargetResource_ function to see if the resource is already in desired state. If not, it runs the _Set-TargetResource_ function. The _Get-TargetResource_ is used only when you run the _Get-DscConfiguration_ cmdlet. Any 3rd party configuration manager trying to use DSC for Windows OS configuration management should ideally use the same workflow.

![](/images/4.png)

In the scenario that I showed in the configuration script at the beginning, I first need to test if the folder already exists on the target system.

```powershell
$Params = @{
   Name = 'File'
   Property = @{'DestinationPath'='C:\DemoRefreshMode'; Type = 'Directory'; Ensure = 'Present'}
   Verbose = $true
}

$TestResult = Invoke-DscResource @Params -Method Test
If (-not $testResult.InDesiredState) {
   Invoke-DscResource -Method Set @Params
}
```

![](/images/5.png)

In the verbose output, we see &#8216;DirectResourceAccess&#8217; as the configuration name. Since we don&#8217;t have a configuration document where a configuration name is defined, DSC internally adds &#8216;DirectResourceAccess&#8217; as the configuration name and calls the CIM method. There is no way to target this cmdlet to a remote system. There is no -ComputerName or -CimSession parameter which means the cmdlet needs to be executed either locally on a target system or use the Invoke-Command cmdlet.

```powershell
$ScriptBlock = {
   $Params = @{
      Name = 'File'
      Property = @{'DestinationPath'='C:\DemoRefreshMode'; Type = 'Directory'; Ensure = 'Present'}
      Verbose = $true
   }


   $TestResult = Invoke-DscResource @Params -Method Test

   If (-not $testResult.InDesiredState) {
      Invoke-DscResource -Method Set @Params
   }
}

Invoke-Command -ComputerName WMF5-2 -ScriptBlock $ScriptBlock
```

This requires PowerShell remoting. In case, you do not have PowerShell remoting enabled, you can directly call the CIM method in the PSDesiredStateConfiguration namespace. The MSFT_DscLocalConfigurationManager class contains three new methods -- ResourceGet, ResourceSet, and ResourceTest. These three methods can be invoked over a CIM session to perform the same task as the Invoke-DscResource cmdlet.

![](/images/6.png)

These CIM methods require more explanation and I will talk about it in a separate article. Stay tuned!

[1]: /2014/10/02/partial-dsc-configurations-in-windows-management-framework-wmf-5-0/