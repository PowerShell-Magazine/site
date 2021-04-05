---
title: Eliminating Redundant Code by Writing Reusable DSC Configurations
author: Mike Robbins
type: post
date: 2015-07-07T16:00:16+00:00
url: /2015/07/07/eliminating-redundant-code-by-writing-reusable-dsc-configurations/
views:
  - 13536
post_views_count:
  - 2506
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
Both PowerShell and DSC (Desired State Configuration) are designed for efficiency through automation. If you’re not careful though, you’ll find yourself bringing the bad habit of repeating yourself performing the same task over and over again from the GUI to PowerShell and DSC by writing the same or similar code over and over. Not only will you be creating redundant code, but you’ll also have to maintain it.

Earlier this year, one of my customers began their hardware and software refresh cycle and my job as an infrastructure architect was to configure both the physical and virtual servers in their datacenters.

Although I had been working with DSC in a test lab environment for quite a while and was comfortable configuring their production systems with DSC, my initial configurations for their primary datacenter contained a lot of redundancy. Due to their timeframe for the project, I found myself hard coding values and creating a separate static DSC configuration for each of their servers. If a server had more than one network card, Windows feature, or service, I had each of those items statically configured in the configuration as well which created redundant code within each of the configurations.

While the configurations shown in this article aren’t the ones used for my customer’s environment, the ones provided will help you to understand how my configurations were written. All of these configurations shown in this article are written for servers running Windows Server 2012 R2 with PowerShell 4.0. The xNetworking DSC resource used in these configurations can be downloaded from GitHub: <https://github.com/PowerShell/xNetworking>. The “x” prefix means it’s experimental.

In this scenario, Server01 is a file server and Server02 is a web server. These configurations are simplistic because complexity isn’t required to understand this concept. Notice that these configurations are different when it comes to the Windows features, services, and the number of network cards.

DSC configuration for Server01:


```powershell
Configuration Server01Config {
     Import-DscResource -ModuleName PSDesiredStateConfiguration, xNetworking
     node Server01 {
          WindowsFeature File-Services {
              Name = 'File-Services'
              Ensure = 'Present'
      	  }
          WindowsFeature FS-FileServer {
              Name = 'FS-FileServer'
              Ensure = 'Present'
          }
          xIPAddress IPNIC1 {
              IPAddress = '192.168.29.171'
              InterfaceAlias = 'Ethernet 2'
              DefaultGateway = '192.168.29.1'
              SubnetMask = '24'
              AddressFamily = 'IPv4'
          }
          xDNSServerAddress IPNIC1 {
              Address = '192.168.29.10', '192.168.29.11'
              InterfaceAlias = 'Ethernet 2'
              AddressFamily = 'IPv4'
              DependsOn = '[xIPAddress]IPNIC1'
         }
    }
}
```
Create the MOF (Managed Object Format) file for Server01:

Server01Config


![](/images/dscreuse1.jpg)

For simplicity, the configurations in this article are being applied via DSC push mode.

Apply the configuration to Server01:

```powershell
Start-DscConfiguration -Wait -Path .\Server01Config –Verbose
```


![](/images/dscreuse2.jpg)

Configuration for Server02:

```powershell
Configuration Server02Config {
     Import-DscResource -ModuleName PSDesiredStateConfiguration, xNetworking
     node Server02 {
        WindowsFeature Web-Server {
            Name = 'Web-Server'
            Ensure = 'Present'
        }
        WindowsFeature Web-Asp-Net45 {
            Name = 'Web-Asp-Net45'
            Ensure = 'Present'
        }
        Service W3SVC {
            Name = 'W3SVC'
            StartupType = 'Automatic'
            State = 'Running'
            DependsOn = '[WindowsFeature]Web-Server'
        }
        xIPAddress 'IPNIC1' {
            IPAddress = '192.168.29.172'
            InterfaceAlias = 'Ethernet'
            DefaultGateway = '192.168.29.1'
            SubnetMask = '24'
            AddressFamily = 'IPv4'
       }
       xDNSServerAddress 'IPNIC1' {
            Address = '192.168.29.10', '192.168.29.11'
            InterfaceAlias = 'Ethernet'
            AddressFamily = 'IPv4'
            DependsOn = '[xIPAddress]IPNIC1'
       }
       xIPAddress 'IPNIC2' {
            IPAddress = '192.168.29.173'
            InterfaceAlias = 'Ethernet 2'
            DefaultGateway = '192.168.29.1'
            SubnetMask = '24'
            AddressFamily = 'IPv4'
       }
       xDNSServerAddress 'IPNIC2' {
            Address = '192.168.29.10', '192.168.29.11'
            InterfaceAlias = 'Ethernet 2'
            AddressFamily = 'IPv4'
            DependsOn = '[xIPAddress]IPNIC2'
      }
   }
}
```

Create the MOF file for Server02:

```powershell
Server02Config
```


![](/images/dscreuse3.jpg)

Apply the configuration to Server02:

```powershell
Start-DscConfiguration -Wait -Path .\Server02Config –Verbose
```

![](/images/dscreuse5.png)

As you can see, the code in the previous two configurations is very redundant because each Windows feature and network card is listed individually and both configurations are accomplishing similar tasks, just for different Windows features and the number of network cards.

Also, notice how much code redundancy that adding just two network cards creates in a single configuration as shown in the previous example for Server02. Now imagine how much redundant code would exist for a physical Hyper-V host virtualization server with eight or more network cards.

When the same company started the hardware and software refresh cycle for one of their secondary datacenters, I began with a copy of the configurations from the systems in their primary datacenter and I spent my allotted time refactoring the configurations to eliminate redundant code and designing a configuration that was generic enough to be used across all of their systems. This involved separating what’s known as the structural configuration (What) from the environmental configuration (Where).

The structural portion of the previously created configurations are close enough that the specifics of each server’s environmental configuration can be removed so that one structural configuration can be used for both. Things like the features and network cards can be run through a foreach loop to eliminate the redundant code within the structural configuration:

Structural Configuration:

```powershell
configuration ServerConfig {
    Import-DscResource -ModuleName PSDesiredStateConfiguration, xNetworking
       node $AllNodes.NodeName {
          $Node.WindowsFeature.ForEach({
             WindowsFeature $_ {
                Name = $_
                Ensure = 'Present'
             }
          })
       $Node.Service.ForEach({
          Service $_.Name {
             Name = $_.Name
             StartupType = 'Automatic'
             State = 'Running'
             DependsOn = $_.DependsOn
         }
      })
      $Node.NIC.ForEach({
         xIPAddress $_.ID {
             IPAddress = $_.IP
             InterfaceAlias = $_.Adapter
             DefaultGateway = $_.Gateway
             SubnetMask = $_.SubnetMask
             AddressFamily = $_.Family
         }
         xDNSServerAddress $_.ID {
             Address = $_.DNS
             InterfaceAlias = $_.Adapter
             AddressFamily = $_.Family
             DependsOn = $_.DependsOn
         }
      })
   }
}
```


The environmental configuration for both of these servers could be placed into a hash table and stored in a variable or PSD1 file. After working with this for a while, I’ve started to prefer storing the environmental configuration for each server into its own PSD1 file so I don’t mistakenly modify and inadvertently break the configuration for a server that I’m not currently working on which wouldn’t necessarily show up until a new configuration for the affected server was created. Storing each one in a separate PSD1 file also helps to keep track of the changes easier when they’re stored in source control.

Environmental configuration for Server01:

```powershell
@{
     AllNodes = @(
        @{
            NodeName = 'Server01'
            WindowsFeature = 'File-Services', 'FS-FileServer'
            NIC = @(
                      @{
                           ID = 'IPNIC1'
                           IP = '192.168.29.171'
                           Adapter = 'Ethernet 2'
                           Gateway = '192.168.29.1'
                           SubnetMask = '24'
                           Family = 'IPv4'
                           DNS = '192.168.29.10', '192.168.29.11'
                           DependsOn = '[xIPAddress]IPNIC1'
                       }
            )
       }
   )
}
```


Environmental configuration for Server02:

```powershell
@{
     AllNodes = @(
          @{
               NodeName = 'Server02'
               WindowsFeature = 'Web-Server', 'Web-Asp-Net45'
               Service = @(
                         @{
                               Name = 'W3SVC'
                               DependsOn = '[WindowsFeature]Web-Server'
                          }
               )
               NIC = @(
                     @{
                          ID = 'IPNIC1'
                          IP = '192.168.29.172'
                          Adapter = 'Ethernet'
                          Gateway = '192.168.29.1'
                          SubnetMask = '24'
                          Family = 'IPv4'
                          DNS = '192.168.29.10', '192.168.29.11'
                          DependsOn = '[xIPAddress]IPNIC1'
                      }
                      @{
                          ID = 'IPNIC2'
                          IP = '192.168.29.173'
                          Adapter = 'Ethernet 2'
                          Gateway = '192.168.29.1'
                          SubnetMask = '24'
                          Family = 'IPv4'
                          DNS = '192.168.29.10', '192.168.29.11'
                          DependsOn = '[xIPAddress]IPNIC2'
                      }
               )
           }
     )
}
```


Create the MOF files for both Server01 and Server02:

```powershell
Get-ChildItem -Path .\PSD1 | ForEach-Object {ServerConfig -ConfigurationData $_.FullName}
```


![](/images/dscreuse6.jpg)

Apply the configuration to both Server01 and Server02:

```powershell
Start-DscConfiguration -Wait -Path .\ServerConfig –Verbose
```


![](/images/dscreuse7.jpg)

Want to know if this process creates the same configuration as the more static version that was used earlier in this article? Produce the MOF configuration files both ways and compare them.

The different methods of creating DSC configurations as shown in this article are simply a means to an end. They’re a way to create the MOF configuration files and it’s those files that are used to apply a configuration to a server. Once the MOF files are created, you won’t need the configurations again until you’re ready to make a configuration change. However, you do need to keep track of your configurations so you know which one is current and so that you’ll have a good starting point when you’re ready to make a change. For this reason, I recommend placing your configurations in some type of source control system. It will make your life so much easier when you don’t make any changes for six months and then you’re trying to figure out where you stored those configurations scripts.

µ