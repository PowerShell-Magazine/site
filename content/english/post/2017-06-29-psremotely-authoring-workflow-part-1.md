---
title: PSRemotely – Authoring workflow (Part 1)
author: Deepak Dhami
type: post
date: 2017-06-29T16:00:00+00:00
url: /2017/06/29/psremotely-authoring-workflow-part-1/
views:
  - 14366
post_views_count:
  - 5205
categories:
  - Pester
  - DevOps
  - Module Spotlight
tags:
  - DevOps
  - Pester
  - Modules

---
### Introduction

After setting up the [context in the previous post][1], it is time to look at how the authoring workflow looks like when using Pester for writing operations validation tests, to begin with, and then leveraging PSRemotely DSL to target it to the remote nodes.

This workflow consists of below stages:

  1. Getting your tests ready, target/test a single node.
  2. Prepare configuration data (abstract hardcoded values).
  3. Use PSRemotely for remote operations validation.
  4. Debugging failures.
  5. Reporting.

> Note – Stages 1-3 will be covered in this post and there will be another post on Stages 4 & 5.
>

Since [PSRemotely][2] was born out of needs for validating an engineered solution, it excels at validating solutions e.g. where the nodes are consistent in behavior and have to be tested for the similar configurations.

### Scenario

To illustrate the point, I am taking an example of deploying the [Hyper-converged solution using Storage Spaces Direct][3]. Now as per the referenced article the deployment has three stages:

  1. Deploy Windows Server
  2. Configure the Network
  3. Configure Storage Spaces Direct

Ideally, the operations validation should run after each step to validate that the entire solution is being configured as per the best practice. To keep today’s post simple, we will be validating only the first step which deploying Windows server but the similar steps apply while authoring the validation tests for the other stages in the deployment workflow.

Now take a look at the referenced link and gather the list of configurations that need to be in place on each node as per the step 1.

  1. Deploy Windows Server 2016.
  2. Verify the domain account is a member of the local administrator group.

So now we have the configurations we need to check on each node just before we configure networking on top of them. You can follow the commits on this [branch][2] on this test repository to see the changes made as part of the authoring workflow.

### Stage 1 – Get your tests ready

This stage consists of authoring tests using Pester/PoshSpec for operations validations.

Let us start by translating the above gathered configurations in Pester Describe blocks independently, to begin with.

Below is a very crude way that can be used to determine that Windows Server 2016 is installed on the node. There are two Pester assertions &#8211;. First one asserts that OS type is a server and the OS SKU is either datacenter edition with GUI or server core.

```powershell
# Ensure that Server 2016 OS installation is done.
Describe "Operating system installed validation" {
    $OS = Get-CimInstance -Class Win32_OperatingSystem

    Context "Server Type check" {
        It "Should have the OSType as WinNT" {
            $OS.OSType | Should Be 18
        }
    }

    Context 'Server 2016 check' {
        It "Should have server 2016 installed" {
           $OS.Caption | Should BeLike '*Server 2016*'
        }
    }
}
```

Here is another independent test for validating that the domain account is a member of the local administrators group on a node.

```powershell
# Validate that the domain account is part of the local administrators group on a node.
Describe "Domain account is local administrator validation" {
    $LocalGroupMember = Get-LocalGroupMember -Group Administrators -Member "S2DClusterAdmin" -ErrorAction SilentlyContinue
    It "Should be member of local admins group" {
        $LocalGroupMember | Should NOT BeNullOrEmpty
    }
}
```


### Stage 2 – Prepare node configuration data

If you look at the authored Pester describe blocks to validate the configuration on the nodes, it might use environment specific data hard coded into the tests e.g. domain username in above example.

So we need to now collect all this data which is environment specific and decouple it from our tests.

Start with the below empty configuration data (place it in the EnvironmentConfigData.psd1 file) and start populating it (it follows the DSC style configuration data syntax).

```powershell
@{
    AllNodes = @(
        @{
            # common node information hashtable
            NodeName = '*'; # do not edit
        },
        @{
            # Individual node information hashtable
            NodeName = 'Node1'
        },
        @{
            NodeName = 'Node2'
        },
        @{
            NodeName = 'Node3'
        },
        @{
            NodeName = 'Node4'
        }
    )
}
```


Start by placing them inside the node configuration data with a general thumb rule of mapping common data to common node information hashtable and node specific details to node configuration hashtable.

Now in the previous tests, the only input is the domain user name. So we can add that to common node information hashtable, since the domain user is a member of the local administrators group needs to be validated on all the nodes in the solution. So now the configuration data looks like below:

```powershell
@{    
    AllNodes = @(
        @{
            # common node information hashtable
            NodeName = '*'; # do not edit
            DomainUser = 'S2DClusterAdmin'
        },
        @{
            # Individual node information hashtable
            NodeName = 'Node1'
        },
        @{
            NodeName = 'Node2'
        },
        @{
            NodeName = 'Node3'
        },
        @{
            NodeName = 'Node4'
        }
    )
}
```


### Stage 3 &#8211; Using PSRemotely for remote ops validation

At this stage in the authoring workflow, we have our tests ready along with the environment configuration data in hand. Before using PSRemotely to target all the nodes for deployment readiness we have to ask this question, How do we connect over PSRemoting to these nodes?

  * Are the nodes domain joined?
  * Connect using the DNS name resolution or IPv4/IPv6 addresses for the remote nodes?
  * Connect using the logged in user account or specifying an alternate account?

Based on the answers to the above questions usage with PSRemotely DSL varies a bit and most of them are documented. For this scenario, the DNS Name resolution of the nodes is used (nodes are already domain joined) and the logged in user account will be used to connect to the remote nodes.

Now it is time to wrap our existing operations validation tests inside the PSRemotely DSL. The DSL consists of two keywords **PSRemotely** and **Node**. PSRemotely is the outermost keyword which allows the framework to:

  * Specify that all ops validations tests are housed inside a <filename>.PSRemotely.ps1 file.
  * [Specify the environment configuration data][4] e.g. hashtable/ .psd1 file/ .json file.
  * [Specify credentials to be used to connect to the each node][5] (if required).
  * [Specify a node specific property in the configuration data][6] to be used for initiating the PSRemoting session.
  * [Populate custom variables in the remote node’s execution context][7].

Node keyword is where we target and organize our tests based on some environment specific data, it is very similar to the Node keyword in the DSC. If you would like to [target different validations to nodes based on some configuration data then it can be done using the Node keyword][8].

Getting back to the problem at hand let’s wrap our existing Pester tests inside the PSRemotely DSL. This is straightforward for the problem at hand and looks like below. We can save the contents of below code snippet in a file called S2DValidation.PSRemotely.ps1 (PSRemotely only accepts file with .PSRemotely.ps1 extension).

> Take note of how the hard coded value for domain username (S2DClusterAdmin) from the standalone Pester tests is replaced with node specific configuration data e.g. $Node.DomainUser.

```powershell
# Use the PSRemotely DSL to wrap the existing Pester tests and target remote nodes
PSRemotely -Path "$PSScriptRoot\EnvironmentConfigData.psd1" {
    # All the nodes in the solution are to be targeted
    Node $AllNodes.NodeName {
        # Ensure that Server 2016 OS installation is done.
        Describe "Operating system installed validation" {
            $OS = Get-CimInstance -Class Win32_OperatingSystem
            Context "Server Type check" {
                It "Should have the OSType as WinNT" {
                    $OS.OSType | Should Be 18
                }
            }
            Context 'Server 2016 check' {
                It "Should have server 2016 installed" {
                $OS.Caption | Should BeLike '*Server 2016*'
                }
            }
        }
        # Validate that the domain account is part of the local administrators group on a node.
        Describe "Domain account is local administrator validation" {
          $LocalGroupMember = Get-LocalGroupMember -Group Administrators -Member "$($Node.DomainUser)" -ErrorAction SilentlyContinue
            It "Should be member of local admins group" {
                $LocalGroupMember | Should NOT BeNullOrEmpty
            }
        }
    }
}
```


We are all set and have two files in the directory e.g. EnvironmentConfigData.psd1 and S2DValidation.PSRemotely.ps1, it is finally time to [invoke PSRemotely][9] and give remote operations validation a go.

We can use Invoke-PSRemotely in the current directory to run all the operations validation housed inside it or specify a path to a file ending with *.PSRemotely.ps1 extension.

{{< youtube nsewGyRRiUw >}}

As shown in the above video, for each node targeted a JSON object is returned. In the returned JSON object the property Status is true if all the tests (Describe blocks) passed on the remote node. Tests property is an array of individual tests (Describe block) run on the Remotely node if all the tests pass then an empty JSON object array of _TestResult_ is returned otherwise the Error record thrown by Pester is returned.

![](/images/remotely1.png)

For the node which failed one of the validations, the JSON object looks like below. Individual _TestResult_ will contain more information on the failing tests on the remote nodes.

![](/images/remotely2.png)

For the failed node, we can quickly verify that out of the two validations targeted at the remote node only one is failing.

![](/images/remotely3.png)

Now there could be a lot many reasons on why the operation validations tests on the remote node are failing. In the next post, we will take a look at how to connect to the underlying PSSession being used by PSRemotely to debug these failures.

[1]: http://www.powershellmagazine.com/2017/04/07/psremotely-framework-to-enable-remote-operations-validation/
[2]: https://github.com/DexterPOSH/RemoteOpsValidationLib/tree/design
[3]: https://technet.microsoft.com/en-us/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct
[4]: http://psremotely.readthedocs.io/en/latest/Example-ConfigurationData/
[5]: http://psremotely.readthedocs.io/en/latest/Example-CredentialHash/
[6]: http://psremotely.readthedocs.io/en/latest/Example-PreferNodeProperty/
[7]: http://psremotely.readthedocs.io/en/latest/Example-Custom-Variables/
[8]: http://psremotely.readthedocs.io/en/latest/PSRemotely-Node-Based-Target/
[9]: http://psremotely.readthedocs.io/en/latest/Invoke-PSRemotely/