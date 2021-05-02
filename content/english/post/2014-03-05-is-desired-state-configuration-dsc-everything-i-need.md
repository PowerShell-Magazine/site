---
title: Is Desired State Configuration (DSC) everything I need?
author: Ravikanth C
type: post
date: 2014-03-05T17:00:40+00:00
url: /2014/03/05/is-desired-state-configuration-dsc-everything-i-need/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
My answer is no! At least, not everything I need right now. A few days ago, I tweeted some of my thoughts. I want to use this article to expand on that.

{{< tweet 435043197329215488 >}}

<span style="line-height: 1.5em;">Before I go ahead saying anything else, let me tell you that I am a BIG fan of DSC and its <a href="/2013/07/05/imperative-versus-declarative-syntax-in-powershell/">declarative style</a> of configuration management. </span>DSC is a new entrant in an already crowded space. But, DSC is significant because it is built into the operating system and it is standards based. PowerShell is one method to use DSC. With the initial release of DSC, product team at Microsoft did a great job. This is certainly a great step forward in realizing the Cloud OS vision. Given the technologies it depends on, it is fair to say that, at some point in time, we would be able to manage non-Windows systems too with DSC.

With PowerShell and DSC, anything that can be scripted can be a DSC resource module. But, that is not always the right approach. With the declarative style, everything looks really clean and simple. But, remember, there is a resource module that is doing that heavy lifting behind the scenes. I have been reading, <a href="/tag/powershell-dsc/">writing</a>, and speaking about DSC. I have also written a few custom DSC resources. Every time, I start writing a DSC resource, I ask myself one question &#8211; do I really need this as a DSC resource? The answer to this is important. Let&#8217;s explore.

### Orchestration vs Configuration Management

<span style="line-height: 1.5em;">If you look deep into the way DSC works, you will see that you either push or pull the configuration. The <a href="http://technet.microsoft.com/en-us/library/dn249922.aspx">Local Configuration Manager</a> (LCM) knows how to tackle the incoming configuration items provided it has all the dependent resource modules. And, the actual configuration change takes place locally. The system that you use either to push or pull configuration is just an intermediate node and does not involve in any sort of coordination. The configuration changes that happen are within the box. C</span><span style="line-height: 1.5em;">onfiguration management (CM) is about standardizing and enforcing </span><span style="line-height: 1.5em;">configurations. This is what </span><span style="line-height: 1.5em;">DSC (and others in this space) is good at. Using configuration documents you can define configuration of a node in a standardized manner and using the configuration mode settings, you can enforce that configuration. A configuration item can be anything that can be changed locally. For example, in DSC, you can use <a href="http://technet.microsoft.com/en-us/library/dn282132.aspx">Package</a> resource to deploy an application or use <a href="http://technet.microsoft.com/en-us/library/dn282133.aspx">Registry</a> resource to add/modify/delete Registry entries.</span>

But, what happens if there is a requirement such that you want to wait for service A on server A to be enabled before you start service B and server B. This calls for _orchestration_.

> From [Wikipedia][1]:
> 
> Orchestration describes the automated arrangement, coordination, and management of complex computer systems, [middleware][2], and services.

This orchestration may include things such as waiting for services on the remote systems, waiting for remote systems to reboot, waiting for install process to complete on remote systems, and so on. There is [no inter-node dependency][3] that can be described in a DSC configuration document. You can, of course, add whatever is required for the orchestration as another DSC resource module. This approach can be quite complex given the semantics of writing a DSC resource module.

I strongly believe that orchestration is a layer above configuration management. There are many existing CM frameworks that provide orchestration capabilities. But, there is nothing close to what a real orchestrator provides. DSC is no exception. Today, we might be able to use products such as the System Center suite to build the orchestration capabilities while leveraging DSC for configuration management and I believe that is the right approach for a larger deployments. But, for small and medium businesses, having the orchestration capabilities built into a feature like DSC can be quite valuable.

### Right tools for the job

<span style="line-height: 1.5em;">When PowerShell 3.0 was released, the </span><a style="line-height: 1.5em;" href="http://technet.microsoft.com/en-us/library/jj134242.aspx">Workflows</a><span style="line-height: 1.5em;"> was touted to be the flagship feature. It was, no doubt about that. Workflows combine the power what background jobs and remoting can do along with lot of other features such as persistence and enabling dependencies through parallel and sequential activities. The biggest problem with Workflows was (as still is), the lack of supported activities. For example, if I want to use some custom cmdlets in a Workflow activity, I have to resort to an InlineScript which almost kills all Workflow features inside it. But, given all the limitations, Workflows can enable </span><strong style="line-height: 1.5em;">orchestration </strong><span style="line-height: 1.5em;">of your data center infrastructure. When I contrast DSC with Workflows for orchestration, I prefer Workflows.</span>

DSC may or may not get the real orchestration capabilities in future. So, at the moment, consider alternate approaches when automating complex multi-machine configurations that are inter-dependent. These alternate approaches can either be standalone scripts or Workflows.

If you are an IT Pro who is just using the custom resources written by someone else, ensure that it satisfies your requirements. For example, when creating VMs on a Hyper-V cluster, the abstracted resource must be the cluster. Without that, when a VM moves from one cluster node to another, DSC will fail to find the VM on the node where the configuration was applied, and it will recreate the VM. This can have unintended consequences.

If you are planning to write custom resources, I suggest that you consider the real usecase and the complexities involved in end to end configuration. This might give you some insight into whether you really need a DSC resource module or not.

All that said, I am still a BIG fan of DSC. I just wish Microsoft adds more capabilities such as [inter-node dependencies][3] and interaction with existing Workflows, etc in the upcoming releases. I have created a connect suggestion for the inter-node dependencies. If you think having that capability eases multi-machine deployments and orchestration, go ahead and [vote for it][3].

[1]: http://en.wikipedia.org/wiki/Orchestration_(computing)
[2]: http://en.wikipedia.org/wiki/Middleware_(distributed_applications) "Middleware (distributed applications)"
[3]: https://connect.microsoft.com/PowerShell/feedback/details/828601/dsc-requires-inter-node-dependency-in-the-configuration