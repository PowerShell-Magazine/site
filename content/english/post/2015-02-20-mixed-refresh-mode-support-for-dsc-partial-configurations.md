---
title: Mixed refresh mode support for DSC partial configurations
author: Ravikanth C
type: post
date: 2015-02-20T19:00:42+00:00
url: /2015/02/20/mixed-refresh-mode-support-for-dsc-partial-configurations/
views:
  - 10777
post_views_count:
  - 1659
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
Partial configurations in DSC [allow department-level delegation of target system][1] configuration. This is an important step in enabling collaboration in the enterprise configuration management context. When [partial configurations feature was introduced][1] in WMF 5.0 Preview November 2014, the only supported configuration delivery mode was Pull mode.

![](/images/Depart.png)

With the WMF 5.0 Preview February 2015, mixing refresh modes or configuration delivery modes is now supported. This means a subset of partial configurations can be pushed using the _Start-DscConfiguration_ cmdlet while the remaining partial configurations are pulled from a pull service or SMB file share.

![](/images/72.png)

In this updated scenario, we have the OS configuration fragment getting pushed while the SQL configuration fragment is being delivered over Pull mode.

### Why mix refresh modes?

What is the need for mixed refresh mode support? Here is what I understand from Microsoft team working on this feature. Mixing refresh modes in partial configurations allow you to gradually transition from a Push-based configuration delivery mode in the data center to a Pull-based configuration delivery mode. Another important use-case of this feature is when you are troubleshooting a specific configuration or when a developer is writing custom DSC resources. You can use the Push configuration delivery mode for a partial configuration fragment and then use the resource script debugging feature in WMF 5.0 to understand what is going wrong in the configuration enact process.

I don&#8217;t recommend using this in a production setup. This should be used purely for dev and test purposes.

### How to use mixed refresh modes?

You can refer to my previous article to see the basic syntax for [adding partial configurations to LCM][1]. To support mixed refresh modes and enable something like what I illustrated in the above picture, a new property called RefreshMode is added to PartialConfiguration setting of the LCM. This can be used to tell LCM what RefreshMode  -- Push, or Pull -- to get a specific fragment of the configuration. When specified, this setting at the partial configuration level overrides the RefreshMode setting at the LCM level.

Note that the new RefreshMode setting _Disabled_ is not supported with partial configurations. There is a bug in February preview that allows you to configure _Disabled_ as the RefreshMode setting for a partial configuration.

Going back to the scenario illustrated in the diagram, I want the target node to get the OS configuration from using push while the SQL configuration fragment will be pulled and it depends on the OS configuration fragment.

The following scripts demonstrate the partial configuration fragments -- OSConfig and SQLConfig. To try the following configuration scripts as-is, you will need the resources from the [DSC Resource Kit][2].

### OS Configuration

```powershell
$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
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
           Name = 'WMF5-2'
           DomainName = 'sccloud.lab'
           Credential = $Credential
        }
    }
}
OSConfig -ConfigurationData $ConfigData -Credential (Get-Credential) -OutputPath 'C:\SMBPull'
New-DSCCheckSum -ConfigurationPath 'C:\SMBPull' -OutPath 'C:\SMBPull'
```


In this configuration fragment, I am using the hostname as the nodename. So, the generated MOF will be named as <hostname>.mof. Since this fragment is being pushed the target system, we don&#8217;t have to use the <GUID>.mof format here or need MOF checksum.

### SQL Configuration

```powershell
$ConfigData = @{
   AllNodes = @(
       @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
       @{ NodeName = 'WMF5-2' }
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
           InstanceName = 'SQLDemo'
           SourcePath = 'E:\'
           Features= 'SQLEngine,SSMS'
           SqlAdministratorCredential = $credential
           DependsOn = '[WindowsFeature]NET35'
       }
   }
}
SQLConfig -ConfigurationData $ConfigData -Credential (Get-Credential) -OutputPath 'C:\SMBPull'
New-DSCCheckSum -ConfigurationPath 'C:\SMBPull' -OutPath 'C:\SMBPull'
```


In the case of SQL configuration, we are using the GUID as the node name. The same GUID will be later configured as the ConfigurationID within the LCM settings. This configuration script assumes that you have OS setup files available at D: and SQL setup files at E:.

Here is how the LCM meta-configuration of the target system looks for this scenario.

```powershell
[DSCLocalConfigurationManager()]
configuration PartialConfigDemo
{
    Node WMF5-2
    {
        Settings
        {
            RefreshMode = 'Pull'
            ConfigurationID = 'a5f86baf-f17f-4778-8944-9cc99ec9f992'
            RebootNodeIfNeeded = $true
        }
           ConfigurationRepositoryShare SMBPull
        {
            SourcePath = '\\WMF5-1\SMBPull'
            Name = 'SMBPull'
        }
           PartialConfiguration OSConfig
        {
            Description = 'Configuration for the Base OS'
            ConfigurationSource = '[ConfigurationRepositoryShare]SMBPull'
            RefreshMode = 'Pull'
        }
           PartialConfiguration SQLConfig
        {
            Description = 'Configuration for the SQL Server'
            DependsOn = '[PartialConfiguration]OSConfig'
            RefreshMode = 'Push'
        }
    }
}
PartialConfigDemo
```


As you see in this meta-configuration document, I have set the LCM RefreshMode to Pull. This is the global setting for LCM in general. For each partial configuration I have a RefreshMode setting again. For the OSConfig partial configuration, it is set to _Push_ and _Pull_ for the SQLConfig fragment. Remember that the ConfigurationSource setting within the partial configuration is mandatory if the RefreshMode for that partial configuration is set to Pull. Otherwise, you don&#8217;t need ConfigurationSource setting.

### Enacting configuration with mixed refresh modes

When all partial configurations are set to _Pull_ refresh mode, the configuration gets enacted during a consistency check or by calling the Update-DscConfiguration cmdlet. However, this process is a bit different when using mixed refresh modes. In our scenario, when we the consistency check kicks in or the Update-DscConfiguration is called, LCM finds that SQL configuration can be pulled over but it depends on OS configuration fragment that is set to push. The SQL configuration fragment cannot be enacted unless the OS configuration is complete. If we try to push the OS configuration fragment using the Start-DscConfiguration cmdlet, you will see an error that partial configurations cannot be pushed this way. Instead, you have to first publish the OS configuration fragment to the partial configuration store using the Publish-DscConfiguration cmdlet.

```powershell
Publish-DscConfiguration -ComputerName WMF5-2 -Path C:\SMBPull -Verbose -Force
```

![](/images/8.png)

When this step is complete, the published configuration can be enacted by specifying the _-UseExisting_ switch parameter with the _Start-DscConfiguration_ cmdlet.

```powershell
Start-DscConfiguration -ComputerName WMF5-2 -Verbose -Wait -UseExisting
```


![](/images/9.png)

The above screenshot shows the output of the enact process. If you observe closely, DSC tries to get the remaining partial fragments from the Pull server. Since an updated fragment already exists on my system, it is not pulled again.

[1]: /2014/10/02/partial-dsc-configurations-in-windows-management-framework-wmf-5-0/
[2]: https://gallery.technet.microsoft.com/scriptcenter/DSC-Resource-Kit-All-c449312d