---
title: Partial DSC Configurations in Windows Management Framework (WMF) 5.0
author: Ravikanth C
type: post
date: 2014-10-02T16:00:07+00:00
url: /2014/10/02/partial-dsc-configurations-in-windows-management-framework-wmf-5-0/
post_views_count:
  - 5094
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
Windows Management Framework (WMF) 5.0 September 2014 Preview brought in many changes to Desired State Configuration (DSC). A couple of important changes include [cross-computer synchronization][1] and partial configurations. I had described the cross-computer synchronization in an [earlier article][1]. In today&#8217;s post, I will explain why we need partial configurations and how to use them.

Let us start with a simple description to understand why we need partial configurations. I see a couple of reasons for this. For the purpose of demonstration and explanation, I will use SQL Server deployment and configuration as an example.

### Incremental Configurations

Imagine we are deploying a SQL Server. Assume that we have already installed OS and we are using DSC to perform the post-OS deployment tasks. The first thing we&#8217;d need to do is make the target system a part of AD domain. Within the SQL configuration, we need to install .NET Framework 3.5 and invoke the SQL deployment, and complete SQL instance configuration. Optionally, OS and application service hardening might be required based on IT policies.

![](/images/frag.png)

What you see in the above picture represents the complete configuration of the target system. Also, as you see above, there are specific fragments of configuration. Although SQL installation depends on the OS configuration, we can still treat them as two completely different fragments. However, with the initial release of DSC, there was no way we could send partial configurations to a target system. That is, first send the OS configuration and then send the SQL configuration to the target system. If we send partial fragments of configuration, the most recently enacted configuration will be the one that is managed by DSC and everything else enacted before that cannot be monitored or managed using DSC any more. The workaround with the initial release was to deliver the OS fragment first and then update the same configuration script to add SQL configuration and deliver it again for enactment.

This is the first problem partial configurations in WMF 5.0 solves. Also, being able to deliver fragments of configuration is important for simulating orchestration experience with DSC.

### Delegated Control and Ownership

While incremental configurations and ability to deliver fragments of configuration is important, the real benefit of partial configurations is in enabling delegated control and ownership. In a real world, the OS deployment and base OS configuration is done by folks or teams different from the people who install, configure, and manage SQL instances. Similar to these teams, there may be a security team that takes care of OS and application hardening configuration. With the earlier release, all these teams had to update the same configuration script. While collaboration is good, there is a huge room for errors. This can be solved effectively using partial configurations. Every team owns its configuration and manages it independent of others. In fact, with partial configurations, it is possible for these different teams even have their own pull server for delivering configurations to the target systems. Having multiple pull servers to deliver partial configurations is optional.

![](/images/Depart.png)

**Note:** In the current implementation (WMF 5.0 September 2014 preview), partial configurations are supported only in pull mode of configuration delivery.

Now that we understand the need for partial configurations, let us dive into how to use partial configurations. To start with, you need to understand the LCM settings that are used for DSC partial configuration.

### Partial Configurations in LCM

Before we can deliver and enact a partial configuration fragment to a target system, the LCM on the target system must be made aware of the fragments. This is done by adding all partial configuration names and the respective pull server details to the LCM settings.

![](/images/24.png)

We will look at the new and updated LCM settings in a later post. But, for now we will discuss the _PartialConfigurations_ property. As you see, this property is an array of CIM instances representing partial configurations. Since partial configurations work only in pull mode at the moment, we need to understand how to configure the target system LCM as a pull client. This configuration changed in WMF 5.0. Here is an example of LCM configuration that uses a REST endpoint as a pull server.


    [DSCLocalConfigurationManager()]
    Configuration SQLVMConfig
    {
        Node localhost
        {
            Settings
            {
                RefreshMode = "Pull"
            }
            ConfigurationRepositoryWeb PullSvc1
            {
                Url = 'http://wmf5-1.sccloud.lab:8080/OSConfig/PSDSCPullServer.svc'
                AllowUnsecureConnection = $true
            }
    	}
    }
    
    SQLVMConfig
The first thing you notice here is the new attribute called _DSCLocalConfigurationManager()_. This identifies the configuration script as the meta-configuration. When using this attribute, the LCM settings need to be placed in the _Settings_ class. This resource is represented by the _MSFT_DSCMetaConfigurationV2_ class whereas the _MSFT_DSCLocalConfigurationManager_ class represents the LCM settings in earlier release of DSC. This class is still available and you can use the old style of configuring LCM in WMF 5.0 too.

![](/images/3dsc.png)

The _ConfigurationRespositoryShare_ and _ConfigurationRepositoryWeb_ are used to specify the SMB and REST pull service configuration respectively. For today&#8217;s post, we will limit our discussion to REST-based pull server. With WMF 5.0, you can specify multiple pull server configurations. So, to add one more pull server configuration, we just add another _ConfigurationRespositoryWeb_ resource instance.


    [DSCLocalConfigurationManager()]
    configuration SQLVMConfig
    {
        Node localhost
       {
           Settings
           {
               RefreshMode = "Pull"
           }
           ConfigurationRepositoryWeb PullSvc1
           {
               URL = 'http://wmf5-1.sccloud.lab:8080/OSConfig/PSDSCPullServer.svc'
               AllowUnsecureConnection = $true
           }
    
           ConfigurationRepositoryWeb PullSvc2
           {
               URL = 'http://wmf5-2.sccloud.lab:8080/SQLConfig/PSDSCPullServer.svc'
               AllowUnsecureConnection = $true
           }
       }
    }
    
    SQLVMConfig
Now that we understand how to add pull server configuration, let us look at the partial configuration settings. So, for the purpose of this demonstration, I assume that we have a VM with Windows Server 2012 R2 installed and the network configuration is complete so that it can join an AD domain. We will create two configuration fragments&#8211;OSConfig and SQLConfig. The OS fragment configures the target system as a domain-joined computer and the SQL fragment installs SQL Server bits. Before we deliver this configuration using pull mode, we need to configure LCM so that it is aware of the configuration fragments.


    [DSCLocalConfigurationManager()]
    configuration SQLVMConfig
    {
       Node localhost
       {
          Settings
          {
              RefreshMode = "Pull"
              ConfigurationID = 'a5f86baf-f17f-4778-8944-9cc99ec9f992'
              RebootNodeIfNeeded = $true
          }
          ConfigurationRepositoryWeb PullSvc1
          {
              URL = 'http://wmf5-1.sccloud.lab:8080/OSConfig/PSDSCPullServer.svc'
              AllowUnSecureConnection = $true
          }
    
          ConfigurationRepositoryWeb PullSvc2
          {
              URL = 'http://wmf5-2.sccloud.lab:8080/SQLConfig/PSDSCPullServer.svc'
              AllowUnsecureConnection = $true
          }
    
          PartialConfiguration OSConfig
          {
             Description = 'Configuration for the Base OS'
             ConfigurationSource = '[ConfigurationRepositoryWeb]PullSvc1'
          }
    
          PartialConfiguration SQLConfig
          {
             Description = 'Configuration for the SQL Server'
             ConfigurationSource = '[ConfigurationRepositoryWeb]PullSvc2'
             DependsOn = '[PartialConfiguration]OSConfig'
          }
       }
    } 
    
    SQLVMConfig
As you see in the above code snippet, the _PartialConfiguration_ resource instances are created for both OS and SQL configuration. We also place a dependency on the OS partial configuration before the SQL configuration fragment can be enacted. Notice the value of _ConfigurationSource_ property. It is written the same way we add values to _DependsOn_ property. In **_OSConfig_** and **_SQLConfig_** partial configurations, we have specified different pull server configuration sources. This is in the form of _**[ResourceName]ResourceIdentifier**_ format.

While authoring the partial configurations, the configuration name must match the identifier associated with the partial configurations defined in the LCM. In the above example, the configurations should be named _**OSConfig**_ and _**SQLConfig**_. Once we have the meta-configuration created, we can run the script to generate the meta-configuration MOF. This can be enacted using the _Set-DscLocalConfigurationManager_ cmdlet. Here is how the LCM configuration looks after I updated the meta-configuration.

![](/images/8dsc.png)

We have the necessary LCM configuration complete to deliver partial fragments of configurations. The following sections look at the actual node configuration fragments for OS and SQL configurations. We will start with OS configuration.

**Note:** For the following code demonstration, you will require DSC Resource Kit xComputer and xSqlServerInstall resources. You can download most recent wave of these resources at: <http://gallery.technet.microsoft.com/site/search?f%5B0%5D.Type=Tag&f%5B0%5D.Value=DSC%20Resource%20Kit%20Wave-7&f%5B0%5D.Text=DSC%20Resource%20Kit%20Wave-7>. Also, this article assumes that you have a target system that has all necessary IP configuration to connect to a domain and has access to SQL Server installation bits. Also, this system must have WMF 5.0 Preview September 2014 installed. If you want to use DSC to do that, [check out my earlier article on that][2].

Also, note that in the meta-configuration, I have set the _RebootNodeIfNeeded_ property _$true_. This ensures that the target system reboots if a resource instance configuration requires.

### OS Configuration Fragment

Within the OS configuration, we want the target system to join a domain. This is enough for our demo purpose. Here is how the configuration looks.

    $ConfigData = @{
       AllNodes = @(
          @{ NodeName = "*"; PsDscAllowPlainTextPassword = $true },
          @{ NodeName = 'a5f86baf-f17f-4778-8944-9cc99ec9f992' }
       )
    }
    Configuration OSConfig {
        Param (
           $Credential
        )
        Import-DscResource -ModuleName xComputerManagement
    
        Node $AllNodes.NodeName {
           xComputer ADJoin {
              Name = 'WMF5-3'
              DomainName = 'sccloud.lab'
              Credential = $Credential
           }
        }
    }
    
    OSConfig -ConfigurationData $ConfigData -Credential (Get-Credential) -OutputPath 'C:\Program Files\WindowsPowerShell\DscService\Configuration'
    
    New-DSCCheckSum -ConfigurationPath 'C:\Program Files\WindowsPowerShell\DscService\Configuration' -OutPath 'C:\Program Files\WindowsPowerShell\DscService\Configuration'
The above configuration script adds the target system to a domain. We are using plain-text credentials in the configuration. Note that this is only for testing purpose and should not be used in production. The next configuration fragment we need is the SQLConfig which installs .NET Framework 3.5 and SQL Server software.

### SQL Configuration Fragment

In the SQL configuration fragment, we want to install .NET Framework 3.5 and then proceed to installing SQL Server software. Here is the configuration script for this.

    $ConfigData = @{
       AllNodes = @(
          @{ NodeName = "*"; PsDscAllowPlainTextPassword = $true },
          @{ NodeName = 'a5f86baf-f17f-4778-8944-9cc99ec9f992' }
       )
    }
    
    Configuration SQLConfig {
        Param (
           $Credential
        )
        Import-DscResource -ModuleName xSqlPS
        Node $AllNodes.NodeName {
            WindowsFeature NET35 {
               Name = 'NET-Framework-Core'
               Source = 'D:\Sources\Sxs'
               Ensure = 'Present'
            }
    
            xSqlServerInstall SQLInstall {
               InstanceName = "SQLDemo"
               SourcePath = "E:\"
               Features= "SQLEngine,SSMS"
               SqlAdministratorCredential = $credential
               DependsOn = '[WindowsFeature]NET35'
            }
        }
    }
    
    SQLConfig -ConfigurationData $ConfigData -Credential (Get-Credential) -OutputPath 'C:\Program Files\WindowsPowerShell\DscService\Configuration'
    
    New-DSCCheckSum -ConfigurationPath 'C:\Program Files\WindowsPowerShell\DscService\Configuration' -OutPath 'C:\Program Files\WindowsPowerShell\DscService\Configuration'
In the above configuration script, I have given the external source path for .NET Framework bits and SQL Server. You need to replace this with whatever is applicable on your target system. Also, you need to pass the credentials that need to be used as the SQL administrator credentials. The _xSqlServerInstall_ resource has many other properties but this is the minimum that is required.

With the above configuration fragment scripts, we have also generated the MOF and the checksum files. These files are generated in the form of _GUID.mof_ and _GUID.mof.checksum _where GUID is the value of LCM _ConfigurationID_ property on the target system. While this works with normal pull server configuration, we need something more with partial configurations. We need to prefix these files with the configuration name mentioned in the LCM _PartialConfigurations_ property. So, in my demo, this is how it looks.

![](/images/2dsc.png)

Without this prefix, the pull client would not be able to identify and download the partial configurations. Once you have all these ingredients ready, we can either wait for the LCM pull client refresh internal to kick in or manually invoke the configuration check. The latter is what I did. This can be done by calling either _PerformRequiredConfigurationChecks_ CIM method of the LCM or using _Update-DscConfiguration_ cmdlet.

```
Invoke-CimMethod -Name PerformRequiredConfigurationChecks -Namespace root/Microsoft/Windows/DesiredStateConfiguration -Arguments @{Flags=[Uint32]2} -ClassName MSFT_DscLocalConfigurationManager -Verbose

#Or the DSC cmdlet
Update-DscConfiguration -ComputerName WMF5-3
```

This method starts a consistency check on the target system which triggers partial configuration download. Since we configured the LCM to reboot as needed, the target system restarts after joining an AD domain. The SQL configuration starts as soon as the target system reboots and completes after a while. You can monitor DSC event logs to check the status of configuration.

Once the configuration is complete, I used the _Trace-xDSCOperation_ cmdlet in the _xDSCDiagnostics_ module to understand the sequence of events during the configuration run.

![](/images/3dsc1.png)

The partial configurations get download to the C:\Windows\System32\Configuration\PartialConfigurations folder. Once all partial configurations are successfully enacted, they get merged into a single configuration represented by current.mof.

It is also possible to incrementally update the LCM partial configurations instead of providing them upfront. This is useful in an orchestration scenario. In this case, the LCM enacts the first partial configuration it receives and copies that as current.mof. At this moment, you can update the LCM meta-configuration to include the remaining partial configurations which will be enacted during the next consistency check.

[1]: /2014/09/26/inter-node-dependency-and-cross-computer-synchronization-with-dsc/
[2]: /2014/09/29/installing-wmf-5-0-preview-september-2014-using-dsc-xwindowsupdate-and-xpendingreboot-resources/