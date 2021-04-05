---
title: 'DevOps, Infrastructure as Code, and PowerShell DSC: Infrastructure Blueprints'
author: Ravikanth C
type: post
date: 2016-05-13T16:00:34+00:00
url: /2016/05/13/devops-infrastructure-as-code-and-powershell-dsc-infrastructure-blueprints/
views:
  - 39485
post_views_count:
  - 4365
categories:
  - DevOps
  - PowerShell DSC
tags:
  - DevOps
  - PowerShell DSC

---
This part of the series is long due after the [first post][1] on DevOps, Infrastructure as Code, and PowerShell DSC. Thanks to PowerShell Conference Europe, I was able to complete what I wanted to publish on GitHub as an example for this article. My session slides and recording should be up in a few days.

Before we see what are infrastructure blueprints, let us do a quick recap of what we discussed in the previous post. Within the infrastructure as code practice, we have source/version control for the reusable automation we need for infrastructure deployment and configuration. In the context of DSC, this reusable automation will be a configuration document and the associated DSC resource modules. The DSC resource modules that are needed for the configuration enact process are a prerequisite for the reusable automation. This enables what we called configuration as code in the previous post.

One of the most important aspects of infrastructure as code is the testing.

**Unit tests** are performed to ensure that the imperative automation behind PowerShell DSC configurations is valid.

Once you enact the configuration, you should always validate the desired state. This can be done using **_integration tests_** written in Pester.

Once the desired state is validated, you must perform **_operations validation_** tests to ensure that the infrastructure is functional and working as expected.

As an infrastructure developer or an administrator responsible for infrastructure configuration, you always should treat the configuration and the tests associated with that as a single entity. This also makes it easy to share these are reusable patterns. This practice is what I call infrastructure blueprints. Let&#8217;s see what exactly these blueprints are?

If you are a developer, you must have read or heard about or might even be practicing _design patterns_. These design patterns are basically solutions to most common problems in application development. These patterns provide **_reusable designs_**. In the infrastructure world, we call these design patterns _reference architectures_. A reference architecture provides a solution template for a given infrastructure requirement. For example, you will find reference architecture papers for implementing a 100,000 Exchange mailbox solution. These reference architectures provide recommendations on choosing the right hardware for a given workload requirement and includes best practices in deploying the application workload. To a large extent, these reference architectures only provide theoretical guidance. By theory I mean that they don&#8217;t really provide the how part of the reference architecture implementation but only the what part. The how part of the reference architecture is usually done either manually or, if you are smart enough, in an automated manner.

Since a complete infrastructure deployment automation example can be very complex, I chose to demonstrate this idea of infrastructure blueprints with a much smaller example of PowerShell DSC and associated tests.

Now, let&#8217;s dig into this in the context of PowerShell DSC by taking a sample scenario.

![](/images/iac1.png)

What is shown in the above diagram is a simple Switch Embedded Team (requires Server 2016) which is used to implement converged networking in Hyper-V. So, within the cluster that I have, I want the converged network switch configuration to be consistent across all nodes in the cluster. Also, I have a set of tests that I want to perform after the resource configuration is complete. This includes both integration and operational validation tests. To be able to share this with my team, I&#8217;ve packaged the DSC configuration into a composite configuration and attached the tests with that. This is my infrastructure blueprint for a Hyper-V converged network switch. I can take this and integrate it into a continuous integration/release pipeline and continuously deploy it in a consistent manner.

Here is how that blueprint looks from a packaging point of view.

![](/images/iac2.png)

You can use the _NodeData.ps1_ to supply the configuration data specific to your environment.

The integration tests listed in ConvergedSwitch.tests.ps1 validate that the resources are in desired state or not.

The _Simple_ and _Comprehensive_ tests in _ConvergedSwitch.Simple.tests.ps1_ and _ConvergedSwitch.comprehensive.tests.ps1_ validate the functionality of the system after the configuration is in desired state. This aligns with the idea of [Operation Validation Framework][2] (OVF). We will see more about operations validation in a later post and see what are some limitations and my thoughts on the future of OVF.

You may call this something else (and not infrastructure blueprint) but you get the story behind that name and understand what they are.

I created a [GitHub repository][3] to share more infrastructure blueprints and associated tests. Feel free to contribute to this with what you have.

[1]: http://powershellmagazine.com/2016/01/05/devops-infrastructure-as-code-and-powershell-dsc-the-introduction/
[2]: https://github.com/PowerShell/Operation-Validation-Framework
[3]: https://github.com/rchaganti/InfraBlueprints