---
title: Inter-node dependency and cross-computer synchronization with DSC
author: Ravikanth C
type: post
date: 2014-09-26T16:00:18+00:00
url: /2014/09/26/inter-node-dependency-and-cross-computer-synchronization-with-dsc/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
I had earlier ranted about why [DSC is not an orchestrator][1] and that I posted a connect suggestion to [add inter-node dependency][2] in DSC. If you have been following WMF 5.0 preview releases, this feature came in the September 2014 release. I still believe that end-to-end orchestration is a layer above configuration management. That said, this new feature&#8211;called cross-computer synchronization&#8211;can be effectively used to validate if a dependent configuration on remote system exists before configuring the local system for any resource.

Let us take a hypothetical example. Note that this is not a real-world scenario, but just an example to demonstrate the concepts.

I have two nodes&#8211;WMF5-3 and WMF5-4. I want to create a configuration script for WMF5-3 to start _Windows Audio_ service. But before this, I want to ensure that the _Windows Audio_ service on WMF5-4 is in running state. In the earlier releases of DSC this is not possible. With WMF 5.0 Preview September 2014, there are three new built-in resources that help us with cross-computer synchronization.

  * WaitForAll
  * WaitForAny
  * WaitForSome

We will get into details of each of these resources in a different article. For this post, let&#8217;s use the _WaitForAll_ resource.

![](/images/dscin.png)

Looking at the syntax for the _WaitForAll_ resource, we understand that the _NodeName_, _ResourceName_, and _Credential_ are the mandatory properties. The _NodeName_ property is used to specify the name of the remote system on the which the dependent resource should be configured. The _ResourceName_ property is used to specify a list of resources that should be configured.

Here is a sample configuration script that I created to demonstrate this feature.

    $ConfigData = @{
        AllNodes = @(
            @{ NodeName = "*"; PsDscAllowPlainTextPassword = $true },
            @{ NodeName = "WMF5-3" }
        )
    }
    
    configuration ComputerSyncPUSH
    {
        Node $AllNodes.NodeName {
           WaitForAll AudioSrv
           {
               ResourceName = '[Service]AudioSrv'
               NodeName = 'WMF5-4'
               RetryIntervalSec = 5
               RetryCount = 3
               Credential = (Get-Credential)
           }
           Service AudioSrv
           {
               Name = "AudioSrv"
               State = "Running"
           }
    	}
    
        Node WMF5-4 {
            Service AudioSrv
            {
                Name = "AudioSrv"
                State = "Running"
            }
        }
    }
    ComputerSyncPUSH -ConfigurationData $ConfigData
This is a simple configuration script where we defined that the configuration on node WMF5-3 depends on the _AudioSrv_ configuration on node WMF5-4. This is defined using the _WaitForAll_ resource. Also, observe that we are passing plain-text credentials by specifying _PsDscAllowPlainTextPassword_ property in the DSC configuration data. This is not the best practice but just a quick hack for this demo.

Now, we need to deliver this configuration. In today&#8217;s article, we will see only the push mode of configuration delivery. The pull mode requires more discussion, so let&#8217;s save it for another day.

One thing you should note here is that the configuration MOF files generated using the above configuration script must exist in the same folder. Once the MOF is generated, we can use the _Start-DscConfiguration_ cmdlet to push this configuration. The following screenshot shows only a specific part of the output which is necessary for our discussion.

![](/images/dscin1.png)

As you see in the output, the PSDSCxMachine.psm1 module gets loaded. This module implements the function for performing cross-computer synchronization and the _WaitForAll_, _WaitForAny_, and _WaitForSome_ resources call the functions in this module. Once the module is loaded, remote resource is checked for desired state. If the remote resource is not in desired state, it gets configured. The retry interval and retry count specified in the _WaitForAll_ resource configuration decides how long the remote Local Configuration Manager (LCM) waits before it either declares a failure or success of the overall configuration change.

So, this is how the cross-computer synchronization is done in WMF 5.0 Preview September 2014. We will see more about this feature and other changes in DSC in upcoming articles.

[1]: /2014/03/05/is-desired-state-configuration-dsc-everything-i-need/
[2]: https://connect.microsoft.com/PowerShell/feedback/details/828601/dsc-requires-inter-node-dependency-in-the-configuration