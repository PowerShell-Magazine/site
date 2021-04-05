---
title: Infrastructure Blueprints – Adding Pre-deployment Validations
author: Ravikanth C
type: post
date: 2017-06-28T16:00:22+00:00
url: /2017/06/28/infrastructure-blueprints-adding-pre-deployment-validations/
views:
  - 10973
post_views_count:
  - 4393
categories:
  - DevOps
  - Pester
  - PowerShell DSC
tags:
  - DevOps
  - Pester
  - PowerShell DSC

---
In one of my [earlier articles][1] here, I wrote about the [Infrastructure Blueprints][2] project. Over the weekend, I published an update this project.

  * Renamed Hyper-VConfigurations composite resource module to HyperVConfigurations. This is a breaking change.
  * Added SystemConfigurations composite resource module containing one composite configuration that includes domain join, remote desktop, timezone, and IE enhanced security configurations.
  * Added Pre-Deploy tests under Diagnostics for each composite resource.

Let&#8217;s come to the subject of today&#8217;s post. In any infrastructure that you are deploying, even f you are no automation guy, there will be a set of prerequisite checks you would perform. For example, if your goal is to deploy a switch embedded team for a Hyper-V host configuration, you will have to check for the existence of physical network adapters in the system that you plan to use within the SET configuration. And, there will be many such pre-deployment checks that you need to perform. So, when using these infrastructure blueprints, it is ideal to package the pre-deployment tests as well into the composite resource module itself.

To address this need, I added PreDeploy scripts under diagnostics tests for each composite resource.

![](/images/infra1.png)

The PreDeploy folder is where all my pre-deployment tests are stored. Here is the pre-deployment test script for the SET team.


```powershell
Describe 'Predeploy tests for Hyper-V Deployment with Switch Embedded Teaming and related network Configuration' {
    Context "Operating System verison tests" {
        $OS = Get-CimInstance -Class Win32_OperatingSystem
        It "Should have the OSType as WinNT" {
            $OS.OSType | Should Be 18
        }
        It "Should have server 2016 installed" {
           $OS.Caption | Should BeLike '*Server 2016*'
        }        
    }

    Context 'Network adapters should exist' {
        Foreach ($adapter in $configurationData.AllNodes.NetAdapterName)
        {
            It "Network adapter named '$adapter' should exist" {
                Get-NetAdapter -Name $adapter -ErrorAction SilentlyContinue | Should Not BeNullOrEmpty
            }
        }
    }
}
```
In the above test scripts, we check that the OS version is indeed Windows Server 2016 to ensure SET configuration can be deployed. Also, we check for the presence of physical network adapters listed in the configuration data to ensure that the SET configuration completes with no errors.

![](/images/infra2.png)

The above flow summarizes the deployment workflow. We execute the pre-deployment tests first and perform deployment only once these tests are all successful. Once the deployment is complete, we run either comprehensive or simple operations tests and put the system into operations only when these tests are successful.

Whatever orchestration script or method that you plan to use, putting this workflow into the process will certainly help you build a resilient deployment pipeline.

[1]: http://www.powershellmagazine.com/2017/05/15/infrastructure-blueprints-a-way-to-share-psdsc-configurations/
[2]: https://github.com/rchaganti/InfraBlueprints