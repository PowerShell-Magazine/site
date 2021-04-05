---
title: 'DevOps, Infrastructure as Code, and PowerShell DSC: The Introduction'
author: Ravikanth C
type: post
date: 2016-01-05T17:00:14+00:00
url: /2016/01/05/devops-infrastructure-as-code-and-powershell-dsc-the-introduction/
post_views_count:
  - 10315
categories:
  - Articles
  - DevOps
  - PowerShell DSC
tags:
  - DevOps
  - PowerShell DSC

---
DevOps is the new cool thing (at least in the Windows world) everyone is talking about and the IT professionals (some of us at least) just give a blank stare every time the DevOps discussion comes up. We feel completely out of place. IT professionals are a part of the DevOps practices and not outside of it. With the rise of [web-scale infrastructures][1], it is important for IT organizations to be more agile and efficient to support the ever growing need for flexible infrastructures than a normal IT professional would have ever imagined. This article series intends to shed some light on DevOps and what it really means for the IT professionals like you and me, and finally explains how and where PowerShell Desired State Configuration plays a role. Let us start with the most [trusted definition of DevOps][2]:

> DevOps (a clipped compound of development and operations) is a culture, movement or practice that emphasizes the collaboration and communication of both software developers and other information-technology (IT) professionals while automating the process of software delivery and infrastructure changes. It aims at establishing a culture and environment where building, testing, and releasing software, can happen rapidly, frequently, and more reliably.

Let us keep the collaboration and communication part of this definition out of our discussion here. It is more of a soft-skill and a practice that should be nurtured between the development and operations teams. There are even tools that enforce this communication and collaboration. Let us focus on later parts the DevOps definition that is about automating the process of software delivery and infrastructure changes and building, testing, and releasing software rapidly, frequently, and more reliably. A picture is worth a thousand words. So, here is the technical part of the DevOps definition in a picture!

![](/images/devopsintro1.png)

What this picture depicts is the typical application code flow from development to production. The phases such as continuous integration and continuous delivery ensure that the developed application code is tested and is stable for deployment in production. The tests that run at these phases provide an assurance that the code will run as expected in all sorts of environments (Development, QA, Staging, and Production) where the code gets deployed. One thing you must note here is that for the application code to run **_as expected_** in any of the environments, you must have the same or similar infrastructure configuration. For example, when you start making changes to the application code, the development infrastructure where you perform unit testing of your code must mimic production infrastructure. If the application code in development requires infrastructure configuration changes, these configuration changes must move, along with the application code, from development to other environments as the code gets tested and deployed.

While delivering and deploying the application code in a rapid and efficient manner is important, it is equally important to ensure that the infrastructure where this code gets deployed is dealt the same way we deal with the application code. For reusable, consistent, and rapid deployments of infrastructure, you need automation. When you have automation, you always want to validate what you are doing because there are no humans sitting and watching the deployment as it happens or to click buttons to complete the deployment. And, finally, when something goes wrong, you want to quickly see what changed in your infrastructure automation and rollback those changes when needed.

This is where infrastructure as code comes into picture!

### Infrastructure as Code

It is easy to argue that what I just described is infrastructure automation which you and I as IT professionals have been doing for ages. Right? One thing you must notice here is that **infrastructure as code is not just about code alone**. Infrastructure automation does play a role within infrastructure as code. After all, how else do we create reusable and repeatable infrastructure without automation. Infrastructure as code mandates software development practices in managing IT infrastructure.

![](/images/devopsintro2.png)

The three major components of infrastructure as code are:

  * Version Control – Enables the method track and rollback changes to your infrastructure as needed. There are many tools that you can use here. My favorite has been Git. If you are new to version control and want to get started, take a look at my [Git for IT Professionals][3]
  * Unit/Integration Testing – Enables validation of your infrastructure code within various phases of DevOps pipeline and lets you feel confident about what you are pushing to production. Pester can be used here. We have great series on [getting started with Pester][4].
  * Infrastructure Blueprints – Enable consistent, reusable, and rapid deployment part of infrastructure as code. Configuration as Code is one of the ways to create infrastructure blueprints but not the only option. Tools like Puppet, Chef, and a platform like PowerShell DSC enable Configuration as Code. These configuration management tools or platforms basically enable declarative way of handling infrastructure configuration. Infrastructure blueprints should always go hand-in-hand with unit/integration testing. We will see more about this in a later post.

Since the first two parts of infrastructure as code are already discussed in other article series here on PowerShell Magazine, I will only focus on infrastructure blueprints that enable consistent, reusable, and rapid deployment and what role PowerShell DSC plays in this space. When I say consistent, I don’t mean that the host names and IP addresses should be the same. Rather, the configuration aspects that impact the application behavior should be the same. This can only be achieved if there is a reusable method to repeatedly deploy this infrastructure in any environment – be it Development, QA, Staging, or Production. You need the ability to separate your environmental configuration from the resource configuration to be able to create a reusable deployment method that works across different environments. For example, the IP addresses and hostnames will be different within each environment. The number of frontend VMs that you would run in a staging environment would be different than the number of VMs in production. Once this level of separation is available, you can create a single blueprint for your infrastructure and then reuse it in any environment just by specifying the right environmental configuration. This is where Puppet, Chef, PowerShell DSC, and other configuration management solutions play a role.

### PowerShell DSC

First of all, understand that **PowerShell DSC is not Infrastructure as Code**. It is one of the enablers of infrastructure as code. I mentioned configuration as code and that is where PowerShell DSC comes into play. It enables a declarative way of expressing your infrastructure configuration. Using this declarative syntax, you can create what I referred to as infrastructure blueprints in the previous section. PowerShell DSC supports separation of environmental configuration from structural or resource configuration. You can use the configuration data in your DSC documents to make your infrastructure blueprints reusable. To ensure that your infrastructure blueprints can be deployed in a repeatable and reliable manner, you need unit and integration tests once the deployment is complete. You can use Pester for this purpose. We shall look at using PowerShell DSC as a part of infrastructure as code in-depth in later parts of this series.

In summary, as an IT professional, it is important for you to know what DevOps brings to enterprise data centers and the difference between infrastructure automation and infrastructure as code. This means you **_transform yourself from an Infrastructure Professional to an_** **_Infrastructure Developer_**.

[1]: http://www.webopedia.com/TERM/W/web-scale-it.html
[2]: http://en.wikipedia.org/wiki/DevOps
[3]: http://104.131.21.239/2015/07/13/git-for-it-professionals-getting-started-2/
[4]: http://104.131.21.239/2015/12/01/pester-explained-introduction-and-assertions/