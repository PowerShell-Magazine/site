---
title: Distributed and Flexible Operations Validation Framework – Introduction
author: Ravikanth C
type: post
date: 2019-06-17T16:16:51+00:00
url: /2019/06/17/distributed-and-flexible-operations-validation-framework-introduction/
post_views_count:
  - 3386
views:
  - 3486
categories:
  - DevOps
  - Module Spotlight
  - Garuda
  - Pester
tags:
  - DevOps
  - Modules
  - Pester

---
Operations validation using PowerShell and Pester has been one of my favorite topics and I have both personal and professional interest in this area. I have invested good amount of time experimenting with the existing frameworks and creating a couple of my own. One of my PowerShell Conference EU sessions was around this topic and a new framework that I am developing. This session was well received and there was good amount of interest in this new framework.

In this series of articles, I will write about the need for a new framework and introduce the new framework which I demonstrated at [PowerShell Conference Europe](https://psconf.eu). There is still a lot of work to be done and this series will act as a way to express where I want to see this whole framework and the end goals.

In this part, you will see what are the available options today for operations validation and / or similar use cases and what their limitations are. Towards the end, I will talk about the desired state requirements for a distributed and flexible operations validation framework.

### The Current State

My work with operations validation started back in 2016 when [I had first demonstrated][2] using Pester for validating the functional state of clusters. This first implementation was [tightly coupled with PowerShell DSC resource modules][3] and needed configuration data supplied with DSC configuration documents to perform the operations validations. This model worked very well for infrastructure that was configured using DSC. However, this is not really a generic or flexible framework for running operations validation.

#### Microsoft Operations Validation Framework (OVF)

Around the time I started working on the operations validations bundled with infrastructure blueprints, PowerShell team published an open source version of a [framework meant for operations validation][4]. This implements operational tests bundled with regular PowerShell modules. The cmdlets in the framework can discover the operations validation tests packaged in the installed modules and invoke the same. You can specify a hashtable of values as the test script parameters. This is a distributed test execution model. Tests can be copied to all nodes and invoked on the node. This is certainly what I wanted to start with. But, the tight coupling between the modules and tests is not what I really want. Instead, I want to be able to distribute chosen tests as groups of tests to any node. I could have written a wrapper script around OVF and achieve what I wanted but there are other limitations. 

Packaging tests as modules is an unnecessary overhead. If you have a huge library of tests and you need to determine the tests that run on the remote targets dynamically, you also need to be able to generate modules dynamically. And, then, you need to find a way to distribute those modules among the target nodes and also ensure that these are kept up to date as you update the central repository.

The test parameters are passed as a hashtable and therefore if you need to invoke the tests in an unattended (such as a schedule task) manner, you need to ensure that you have a wrapper script that reads some sort of configuration data and translates that into relevant parameters. But, then you need a way to publish that configuration data as well to the remote targets. 

#### PSHealthz

[PSHealthz][5] by Brandon Olin provides a web service endpoint to invoke tests packaged or published using OVF. This is an implementation of the [Health Endpoint Monitoring Pattern][6] using PowerShell. The available tests can be retrieved using the /health endpoint. Tests can be executed on the target node using the query parameters on the /health endpoint. PSHealthz is more of a way to list and invoke tests on the target nodes using the REST API but the limitations of OVF I mentioned above still exist. 

#### Remotely and PSRemotely

[Remotely][7] is an open source PowerShell module from Microsoft that can be used for running Pester tests remotely &#8212; no surprises there! You can specify a set of remote targets in a file called machineconfi.csv and then use the Remotely keyword inside the It scriptblock for running the tests on the remote targets. This module has several drawbacks and has been more experimental than anything remotely useful (pun intended!). In fact, it has been more than 3 years since there was any update. Although the tests run on the remote node (using PowerShell remoting), they are essentially triggered from a central location in a fan-out method. Therefore, this module implements centralized test execution and reporting.

[PSRemotely][8] was born out of the need for running tests on a bunch of remote nodes while eliminating all of Remotely drawbacks and providing better control over what runs when and where. This module uses DSC type configuration data for providing test parameters for each remote node. In fact, we have implemented a complete validation suite using PSRemotely before writing one more internal framework for operations validation of clusters. The major drawback of this module was the need to enable CredSSP so that the delegated credentials, when needed, can be used on remote targets. Also, there was no infrastructure awareness in PSRemotely. The number and type of tests running gets determined using the configuration data and we had no control over grouping tests based on the type of infrastructure. With PSRemotely, the execution of tests is distributed and reporting is centralized. Therefore, PSRemotely implements a hybrid model. With this framework, Pester tags is the only way to separate tests into groups.

#### DBAChecks and pChecksAD

Both [DBAChecks][9] and [pChecksAD][10] implement a more centralized test execution and reporting. All tests stay on the local system and you can design these tests to target remote systems using a cmdlet provided method or write your tests to use PowerShell remoting to target remote systems. These are purpose built modules but you can take clues from how they implemented these modules and write one for your specific use case. These are great at what they are doing but not something that would satisfy my requirements for a distributed and flexible operations validation framework.

#### PoshSpec

<a href="https://github.com/ticketmaster/poshspec" target="_blank" rel="noreferrer noopener" aria-label="PoshSpec  (opens in a new tab)">PoshSpec </a>is another great project that enables simplified infrastructure validations with a way to extend it very easily. PoshSpec provides a DSL for infrastructure validations for known resources. For example, you can write a test in PoshSpec DSL to verify if a hotfix is installed without worrying about how to get a list of hotfixes. There is currently a <a rel="noreferrer noopener" aria-label="limited set of resources (opens in a new tab)" href="https://github.com/ticketmaster/poshspec/tree/master/Public" target="_blank">limited set of resources</a>. PoshSpec is centralized test execution. It does not yet support remote execution but there is an <a rel="noreferrer noopener" aria-label="RFC on that (opens in a new tab)" href="https://github.com/ticketmaster/poshspec/issues/6" target="_blank">RFC on that</a>. I created this issue back in 2016 and continued to work on my own frameworks for achieving what I really need.

### The Desired State

You have seen, so far, options available for performing operations validation. You have also read about the limitations that these frameworks or modules pose. I will, now, translate these limitations into the requirements for a new operations validation framework.

#### Distributed

The new framework needs to support distribution (publishing) of tests to remote targets and should offer different methods for test distribution. For example, I should be able to publish tests to remote targets using PowerShell DSC or Ansible or Chef or Puppet.

The new framework should support distributed test execution. I want to be able to invoke tests on-demand or on a scheduled basis on the remote targets. The input parameters or configuration data needed for the tests should be local but the framework should provide a way to publish the configuration data as well. And, the secrets within the configuration data should be encrypted.

#### Flexible

The new framework should be flexible enough to allow integration with different other modules or technologies. For example, I had mentioned already that the test distribution should support more than one method. 

Within infrastructure management, there will be more than one team involved in bringing up the infra. For example, if there is a SQL cluster that is being managed, there may be a team that is solely responsible for OS deployment & management whereas another takes care of SQL management. Now, each of these team will have their own operations validation tests. The new framework should enable a way to publish multiple test groups to the remote targets and execute and report them independently. 

From a reporting point of view, the new framework should be capable of supporting multiple reporting methods like HTML, Excel, and so on.

#### Secure

The tests running on the remote targets need input parameters and this may include secure strings and secrets. Since this configuration data needs to reside on the target nodes, the sensitive data should be stored in a safe manner. For example, credentials should go into a vault such as Windows Credential Manager. The new framework should support this.

The result retrieval from the remote targets happens at a central console. For this, the test operators need access only to invoke the test result retrieval from the remote targets. The framework should support least privileged way of doing this such as implementing a JEA endpoint.

### Introducing Garuda

I have been experimenting a bit trying to implement a totally new framework that satisfies most if not all of the desired state requirements. This is still in a [proof-of-concept phase][11]. There is not even documentation around how to use this yet. This is what I demonstrated at the PowerShell Conference EU 2019 a week ago. I named this framework Garuda. I will write about the naming choice in the next post.

Today&#8217;s article is an introduction to the thought process behind Garuda. In the next post, I will explain the architecture of Garuda and talk about how some of the desired state requirements are implemented.

BTW, just before my session at the EU conference, I had a couple of hours to kill and created this logo for the Garuda framework. You will know the meaning of this in the next post.

![image](/images/garudalogo.png)


[1]: http://www.psconf.eu/
[2]: https://www.youtube.com/watch?v=vmEeTBSWd5s
[3]: https://github.com/rchaganti/InfraBlueprints/tree/Dev/HyperVConfigurations
[4]: https://github.com/PowerShell/Operation-Validation-Framework/
[5]: https://github.com/devblackops/pshealthz
[6]: https://msdn.microsoft.com/en-us/library/dn589789.aspx
[7]: https://github.com/PowerShell/Remotely
[8]: https://github.com/DexterPosh/PSRemotely
[9]: https://github.com/sqlcollaborative/dbachecks
[10]: https://github.com/mczerniawski/pChecksAD
[11]: https://github.com/rchaganti/garuda