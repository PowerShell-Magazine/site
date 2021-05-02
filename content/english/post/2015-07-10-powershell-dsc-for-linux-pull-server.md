---
title: 'PowerShell DSC for Linux: Pull Server'
author: Bartek Bielawski
type: post
date: 2015-07-10T16:00:17+00:00
url: /2015/07/10/powershell-dsc-for-linux-pull-server/
views:
  - 11331
post_views_count:
  - 1977
categories:
  - PowerShell DSC
  - Linux
tags:
  - PowerShell DSC
  - Linux
---
Initial release of PowerShell DSC for Linux had one big gap: it supported only pushing the configuration to the node. And even though this mode is good enough for demos or basic proof of concept, it doesn&#8217;t seem like a proper solution for production environment. The first official version of PowerShell DSC for Linux supports both modes of configuration delivery&#8211;Push and Pull. In this part of the series, we will take a look at setting up Linux to pull configuration from single and multiple pull servers.

When we author DSC configuration for Linux, we can use both WMF4 and WMF5. Same applies to meta-configurations used to set up Local Configuration Manager (LCM) settings. In both versions we have to set LCM _RefreshMode_ to _Pull_ and define/list download managers. First, we will take a look at configuration for LCM defined using WMF4 syntax:

```powershell
Configuration LinuxPull {
    param (
        [String]$ComputerName,
        [guid]$Id
    )    
    node $ComputerName {
        LocalConfigurationManager {
            RefreshMode = 'Pull'
            ConfigurationID = $id.Guid
            ConfigurationMode = 'ApplyAndAutocorrect'
            DownloadManagerName = 'WebDownloadManager'
            DownloadManagerCustomData = @{
                ServerUrl =
                  'https://pull.monad.net:8080/PSDSCPullServer/PSDSCPullServer.svc'
            }
            RebootNodeIfNeeded = $true
        }
    }
}
```


Configuration using WMF5 (based on <a href="http://blogs.msdn.com/b/powershell/archive/2015/04/29/windows-management-framework-5-0-preview-april-2015-is-now-available.aspx" target="_blank">April 2015 preview</a>) is slightly different mainly because of separation of global settings and configuration for download managers and attribute that we need to apply to the configuration in order to distinguish it from &#8220;normal&#8221; configuration:

```powershell
[DSCLocalConfigurationManager()]
configuration LinuxPullv5 {
    param (
        [String]$ComputerName,
        [guid]$Id
    )
    node $ComputerName {
        Settings {
            RefreshMode = 'Pull'
            ConfigurationID = $id.Guid
            ConfigurationMode = 'ApplyAndAutocorrect'
        }
        ConfigurationRepositoryWeb main {
            ServerURL = 'https://pull.monad.net:8080/PSDSCPullServer.svc'
        }
    }
}
```


As a next step, we have to generate MOF document for our node and save it to the correct location together with matching checksum file. For testing purposes we will use _hello world_ configuration to avoid any kind of issues with configuration itself:

```powershell
Configuration TestPull {
    param (
        [String]$ComputerName
    )
    Import-DscResource -ModuleName nx
    node $ComputerName {
        nxFile Test {
            DestinationPath = '/tmp/pull'
            Contents = "Hello World!`n"
        }
    }
}
TestPull -ComputerName $id.Guid -OutputPath $mainConfig
New-DscChecksum -Path $mainConfig -Force
LinuxPullv5 -ComputerName PSMag.monad.net -Id $id
Set-DscLocalConfigurationManager -Path .\LinuxPullv5 -CimSession $linuxCim -Verbose
Update-DscConfiguration -CimSession $linuxCim -Wait
cURL failed to perform on this base url: pull.monad.net with this error message: Peer certificate cannot be authenticated with given CA certificates.
```


The result we get should not be too surprising for anybody who worked with Pull servers. We told our Linux node to talk to Pull server using HTTPS. The problem is that we didn&#8217;t do anything to make sure that Linux trusts the certificate on our Pull server. If nodes are only Windows and domain-joined that can be handled easily with group policies. Release notes for PowerShell DSC for Linux suggest to walk around this problem by forcing cURL to trust our certificate. What I would recommend instead is to configure Linux to trust our Enterprise CA (I&#8217;m making assumption that you already use it for certificates on Pull servers). It gives us extra advantage that if we ever decide to change the Pull server, or use more than one, we don&#8217;t have to repeat all the steps necessary to convince cURL to trust it. I suspect procedure may differ between distributions; the one I followed is for CentOS (assuming _monad-ca-ca.cer_ is locally saved certificate file for our enterprise CA):

```powershell
$certFile = @{
    LocalFile = '.\monad-ca-ca.cer' 
    RemotePath = '/etc/pki/ca-trust/source/anchors/' 
    Credential = $root 
    ComputerName = 'PSMag.monad.net'
}
Set-SCPFile @certFile
$ssh = New-SSHSession -ComputerName PSMag.monad.net -Credential $root
$PSDefaultParameterValues.'Invoke-SSHCommand:SSHSession' = $ssh
Invoke-SSHCommand -Command 'update-ca-trust enable'
Invoke-SSHCommand -Command 'update-ca-trust extract'
Invoke-SSHCommand -Command '/opt/omi/bin/ConsistencyInvoker'
Invoke-SSHCommand -Command 'cat /tmp/pull'
```

**Note**: I have used <a href="https://github.com/darkoperator/Posh-SSH" target="_blank">Posh-SSH</a> module to copy files/invoke commands, you can read more about it <a href="/2014/07/03/posh-ssh-open-source-ssh-powershell-module/" target="_blank">here</a>.

If our certificate was added to ca-trust we should be able to see content of our _hello world_ file. If that didn&#8217;t help reading logs and observing behavior of omiserver run interactively may give us clues why it still fails.

I&#8217;ve mentioned using more than one Pull server few times already. There is very good reason for that: PowerShell DSC for Linux not only supports Pull server, it also supports partial configurations. You can read more about partial configurations in one of the previous <a href="/2014/10/02/partial-dsc-configurations-in-windows-management-framework-wmf-5-0/" target="_blank">articles written by Ravi</a>. Long story short: partial configurations enable scenarios where separation of roles and taking pieces of configuration from different Pull servers (potentially owned by different teams) is necessary. The way partial configuration work evolves: when I tried to apply patterns from Ravi&#8217;s post with current version it failed. The procedure I had to follow:

  * create configuration that contains two Pull servers and two partial configurations
```powershell
[DSCLocalConfigurationManager()]
configuration PartialConfiguration {
    param (
        [String]$ComputerName,
        [guid]$Id
    )
    node $ComputerName {
        Settings {
            RefreshMode = 'Pull'
	        ConfigurationID = $id.Guid
            ConfigurationMode = 'ApplyAndAutocorrect'
        }
        ConfigurationRepositoryWeb pull {
          ServerURL = 'https://pull.monad.net:8080/PSDSCPullServer.svc'
        }
        ConfigurationRepositoryWeb partial {
            ServerURL = 'https://partial.monad.net:8080/PSDSCPullServer.svc'
        }
        PartialConfiguration base {
            ConfigurationSource = '[ConfigurationRepositoryWeb]pull'
        }
        PartialConfiguration details {
            ConfigurationSource = '[ConfigurationRepositoryWeb]partial'
            DependsOn = '[PartialConfiguration]base'
        }
    }
}
```


  * create two configurations with the names matching LCM partial configurations and drop generated MOFs to Pull servers (I didn&#8217;t change file&#8217;s&#8217; names)
  * generate checksums for both files
```powershell
Configuration base {
    param (
        [String]$ComputerName
    )
    Import-DscResource -ModuleName nx
    node $ComputerName {
        nxFile test {
            DestinationPath = '/tmp/pull'
            Contents = "I'm from Pull server!`n"
        }
    }
}
base -ComputerName $id.Guid -OutputPath $mainConfig
New-DscChecksum -Path $mainConfig -Force

Configuration details {
    param (
        [String]$ComputerName
    )
    Import-DscResource -ModuleName nx
    node $ComputerName {
        nxFile test2 {
            DestinationPath = '/tmp/partial'
            Contents = "I'm just partial!`n"
        }
    }
}
details -ComputerName $id.Guid -OutputPath $partialConfig 
New-DscChecksum -Path $partialConfig -Force
```

  * configure LCM with previously created meta MOF document

I was using April preview of WMF5, so before doing last step I had to modify generated MOF. My client was saving partial configurations as an array, LCM was expecting singleton. Here is simple function I&#8217;ve used to clean up MOF and code I&#8217;ve used to generate, update and implement it:

```powershell
function Update-MetaDocument {
    param (
        [Parameter(
            ValueFromPipeline,
            Mandatory
        )]
        [ValidateScript({
            Test-Path -Path $_
        })]
        [String]$Path
    )
    process {
        $body = Get-Content @PSBoundParameters -Raw
        $body -replace  '(?s)(ConfigurationSource = )\{([^}]*)};', '$1 $2;' | 
            Set-Content @PSBoundParameters -Force
    }
}
PartialConfiguration -ComputerName PSMag.monad.net -Id $Id | Update-MetaDocument
Set-DscLocalConfigurationManager -CimSession $linuxCim -Path .\PartialConfiguration
```


Note: if you are using February preview of WMF5 cleaning up MOF document shouldn&#8217;t be necessary. Schema for meta-configuration of LCM changed between these two versions and PowerShell DSC for Linux was is currently based on earlier version.

After completing these steps and updating configuration on the Linux node I could see both files on the disk with expected content:

```powershell
[root@PSMag ~]# ConsistencyInvoker
[root@PSMag ~]# cat /tmp/pull
I'm from Pull server!
[root@PSMag ~]# cat /tmp/partial
I'm just partial!
```


As you can see PowerShell DSC for Linux matured a lot since its initial release. I must also say that team is more responsive at the moment. I reported an <a href="https://github.com/MSFTOSSMgmt/WPSDSCLinux/issues/18" target="_blank">issue</a> with the current version (related to Polish characters in the MOF document and using UTF8-encoded MOF) and I was able to apply a patch that fixes this problem few days later. You can find this patch in a <a href="" target="_blank">separate branch</a> on GitHub repo for PowerShell DSC for Linux. For me that change is as important as all the features added in the current release: being able to report problems and receive fixes for them between releases is very important for any early adopter. I would love to be able to contribute to the project and there is opportunity for that too: for now it&#8217;s just resources written in native code (you can read about it in <a href="https://github.com/MSFTOSSMgmt/WPSDSCLinux/tree/Fix-UnicodeStrings" target="_blank">Linux DSC announcement on PowerShell team blog</a>), but I hope it will extend to Python-based resources and the core product soon. Keeping all that in mind I&#8217;m more confident than before that PowerShell DSC for Linux is something we should start to consider as a solution for managing Linux systems.