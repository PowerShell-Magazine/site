---
title: PSRemotely – Framework to Enable Remote Operations Validation
author: Deepak Dhami
type: post
date: 2017-04-07T16:00:40+00:00
url: /2017/04/07/psremotely-framework-to-enable-remote-operations-validation/
views:
  - 13840
post_views_count:
  - 5136
categories:
  - Module Spotlight
  - DevOps
  - Pester
tags:
  - DevOps
  - Pester
  - Modules

---
Before we get started with what is PSRemotely, here is some background.

As part of my work in an engineering team, I am tasked with writing scripts which will validate the **_underlying infrastructure_** before the automation (using PowerShell DSC) kicks in to deploy the solution.

Below are the different generic phases which comprise the whole automation process:

  * **Pre-deployment** – getting the base infrastructure ready, the bare minimum required for automation. For example – network configuration on the nodes is needed.
  * **Deployment** – deployment of the solution leveraging PowerShell DSC.
  * **Post-deployment** – scripts/runbooks configuring or tweaking the environment.

What I meant by validating **_underlying infrastructure_** above, is that the compute and storage physical hosts/nodes have a valid IP configuration, connectivity to the AD/DNS infrastructure etc. the **key** components that we required to be tested and validated to get confidence in our readiness to deploy the engineered solution on top of it.

**Note** – Our solution had scripts in place that would configure the network based on some input parameters and record this in a manifest XML file. After the script ran, we would assume that everything is in place. These assumptions at some points cost us a lot of efforts in troubleshooting.

In short, initial idea was to have scripts validating, what the scripts did in an earlier step. So it began, I started writing PowerShell functions, using workflows (to target parallel execution on nodes). This was a decent solution until there were requests to add validation tests for entirely everything in the solution stack e.g. DNS configuration, network connectivity, proxy configuration, disks (SSD/HDD) attached to the storage nodes etc.

Phew! It was a nightmare maintaining it.

Rays of hope: [Pester][1], [PoshSpec][2], and [Remotely][3]!

We went into looking at how to use some of the open source PowerShell modules into helping us perform operations validation. At this time in community, Pester was gaining traction for the operations validation.

### Using Pester

We moved away from using standalone scripts for the operations validation and started converting our scripts into Pester tests. It is not surprising to see that many operations people find it easier to relate to using Pester for Ops validation, since we have been doing this validation for ages manually. Pester just makes it easy to automate all of it.

For example, in our solution each compute node gets three NIC cards, pre-deployment script configures them. If we had to test whether the network adapter’s configuration was indeed correct, it would look something like below using Pester:




```powershell
Describe "TestIPConfiguration" {
    It "Should have a valid IP address on the Management NIC" {
        (Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias 'vEthernet(Management)' | Select-Object -ExpandProperty IPAddress) |
            Should be '10.10.10.1' 
    }
	It "Should have a valid IP address on the Storage1 NIC" {
   		(Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias 'vEthernet(Storage1)' | Select-Object -	ExpandProperty IPAddress) |
        Should be '10.20.10.1'  
	}

	It "Should have a valid IP address on the Storage2 NIC" {
    	(Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias 'vEthernet(Storage1)' | Select-Object -	ExpandProperty IPAddress) |
        Should be '10.30.10.1'  
}
```
### Using PoshSpec & Pester

PoshSpec added yet another layer of abstraction on our infrastructure tests by adding yet another DSL.

Below is how our tests started looking with usage of Pester and PoshSpec.

**Note** – For validation of IPv4 Address, another keyword named IPv4Address was added to PoshSpec which would essentially call Get-NetIPAddress and spit out the IPv4 address assigned on the NIC interface with specified alias.

```powershell
Describe "TestIPConfiguration" {
    Context "Validate the Management NIC " {
        IPv4Address 'vEthernet(Management)' {Should be '10.10.10.1'} 
    }

   Context "Validate the Storage1 NIC" {
        IPv4Address 'vEthernet(Storage1)' {Should be '10.20.10.1'} 
    }
	Context "Validate the Storage2 NIC" {
    	IPv4Address 'vEthernet(Storage2)' {Should be '10.30.10.1'}
	}
}
```
By using Pester and PoshSpec to write tests, it sure made maintaining these tests easy, but we still have a problem at 

hand.  How do we target our above tests to all the nodes in the solution?

### [Remotely][3]??

At some point [Ravi](https://ravichaganti.com) was [tinkering][5] with this particular PowerShell module and suggested to take a look at it. It was promising to begin with as he added support for passing Credential hash to Remotely. We would have to specify a hash table with the computer name as key and credential as value to Remotely and it would take care of connecting to those nodes, executing the script block in the remote runspace. At this point things started falling in place for what we had in mind. Our tests started looking nice and concise:

```powershell
$CredHash = @{
    'ComputeNode1' = Get-Credential
    'ComputeNode2' = Get-Credential
}
Describe "TestIPConfiguration" {
    Context "Validate the Management NIC " {
       Remotely ComputeNode1, ComputeNode2 {IPv4Address 'vEthernet(Management)' {Should be '10.10.10.1'}} 
    }

   Context "Validate the Storage1 NIC" {
        Remotely ComputeNode1, ComputeNode2 {IPv4Address 'vEthernet(Storage1)' {Should be '10.20.10.1'}} 
    }

    Context "Validate the Storage2 NIC" {
        Remotely ComputeNode1, ComputeNode2 {IPv4Address 'vEthernet(Storage2)' {Should be '10.30.10.1'}} 
    }
}
```

Soon we realized that the Assertions above e.g. {Should Be ’10.10.10.1’} are to be dynamically created by reading the manifest XML file which drives the whole deployment. It contains what is the expected configuration on the remote nodes.

We wanted our tests to be generic so that we could target them to all nodes part of the solution. We were looking to have our tests organized like below, where of course a node-specific details e.g. $ManagementIPv4Address etc. would be read from the manifest file and created on the fly either on the local machine or remote node :

```powershell
$CredHash = @{
    'ComputeNode1' = Get-Credential
    'ComputeNode2' = Get-Credential
}
Describe "TestIPConfiguration" {
    Context "Validate the Management NIC " {
       Remotely ComputeNode1, ComputeNode2 {IPv4Address 'vEthernet(Management)' {Should be  $ManagementIPv4Address}} 
    }

   Context "Validate the Storage1 NIC" {
        Remotely ComputeNode1, ComputeNode2 {IPv4Address 'vEthernet(Storage1)' {Should be $Storage1IPv4Address}} 
    }
    Context "Validate the  Storage2 NIC" {
        Remotely ComputeNode1, ComputeNode2 {IPv4Address 'vEthernet(Storage2)' {Should be $Storage2IPv4Address}}  
    }
}
```
The above syntax looks quite descriptive and decouples the validation tests and environment details too.

But there were some downsides to the above approach.

  * Requires us re-writing our existing tests to accommodate keyword Remotely for executing script block on remote and running assertions locally.
  * Remotely connects each time to all the nodes to run each PoshSpec based ops validation tests. Results in lot of overhead to run a large number of validation tests.
  * Trouble passing environment specific data to the remote nodes e.g. in the above tests passing the expected IPv4 address to the remote node.
  * For running Pester/PoshSpec tests on the remote nodes, these modules need to be present on the remote node, to begin with.

The existing Remotely framework was meant to execute script block against a remote runspace but it was not specifically built to perform operations validation remotely.

### Enter PSRemotely

After trying to integrate Remotely with Pester/PoshSpec based tests, we had a general idea on what we needed from a framework/DSL, if it was to provide us with the capability of orchestrating operations validation remotely on the nodes. Below are some of those features we had in mind along with the arguments for these to be implemented:

  * Target Pester/PoshSpec based operations validation tests on the remote nodes.
  * Allow specifying environment data separately from the tests, so that same tests could be applied across on nodes.
  
    __We decided on the ability to use DSC Style configuration data here for specifying node specific environment details.__
  * Easier debugging on the remote nodes, in case tests fail.
  
    __If something failed on the remote node during validation, we should be able to connect to the underlying PowerShell remoting session and debug issues.__
  * Allow re-running specific tests on the remote nodes.
  
    __In case a test failed, performing a quick remediation action and validating that specific test passed it is a good to have feature when you have lot of tests in your suite.__
  * Self-contained solution.
  
    __Have the framework bootstrap the remote nodes with required modules version (Pester & PoshSpec) under the hood. Remote nodes might not have internet connectivity here.__
  * Allow copying required artifacts to remote nodes.
  
    __For our solution, we require a manifest file with details about the deployment to be copied on each node.__
  * Use PowerShell remoting as underlying transport mechanism for everything.
  * Return bare minimum JSON output, if everything passes. If a test fails then return the error record thrown by Pester.

And, [PSRemotely][6] was born!

So after a lot of discussions with Ravi, I finally got insight on how the remote operations validation DSL should look like:

```powershell
$CredentialHash = @{
    'ComputeNode1' = Get-Credential
    #'ComputeNode2' = Import-CliXML # If a node is missed here, current user creds are used with PSRemoting
}
```

Configuration data, can be generated separately or specified from a .psd1 or .json file

```json
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = '*' 
            DomainFQDN = 'dexter.lab'
        },
        @{
            NodeName = "ComputeNode1"
            ManagementIPv4Address = '10.10.10.1'
            Storage1IPv4Address = '10.20.10.1'
            Storage2IPv4Address = '10.30.10.1'
            Type = 'Compute'
        },
        @{
            NodeName = "ComputeNode2"
            ManagementIPv4Address = '10.10.10.2'
            Storage1IPv4Address = '10.20.10.2'
            Storage2IPv4Address = '10.30.10.2'
            Type = 'Compute'
        }
    )
}
```

PSRemotely tests, specify CredentialHash and Configuration data to the PSRemotely  

```powershell
PSRemotely -ConfigurationData $ConfigData -CredentialHash $CredentialHash {
    Node $AllNodes.Where({$_.Type -eq 'Compute'}).NodeName {
        #Below are the existing Pester/PoshSpec-based tests, with changes on how node specific data is supplied

        Describe 'TestIPConfiguration' -Tag IP {
            Context "Validate the Management NIC" {
                    # pre-deployment script always creates NICs with these names
                    IPv4Address 'vEthernet(Management)' {Should be  $Node.ManagementIPv4Address}
            }

            Context "Validate the Storage1 NIC" {
                IPv4Address 'vEthernet(Storage1)' {Should be $Node.Storage1IPv4Address} 
            }

            Context "Validate the Storage2 NIC" {
                IPv4Address 'vEthernet(Storage2)' {Should be $Node.Storage2IPv4Address}
            }
        }       
	}
}
```
After having a clear idea on the features required in the framework and how we wanted the DSL to look like, I started working on it. This post has set up the context on why we began working on something entirely new from scratch.

Join me in the second post where I try to explain how to use [PSRemotely][6] to target remote nodes for operations validation.

[1]: https://github.com/pester/pester
[2]: https://github.com/Ticketmaster/poshspec
[3]: https://github.com/PowerShell/Remotely
[4]: http://www.powershellmagazine.com/author/ravikanth/
[5]: https://github.com/rchaganti/Remotely
[6]: https://github.com/DexterPOSH/PSRemotely