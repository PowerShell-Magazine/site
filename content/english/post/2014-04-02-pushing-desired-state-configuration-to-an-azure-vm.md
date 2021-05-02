---
title: Pushing DSC configuration to an Azure VM
author: Ravikanth C
type: post
date: 2014-04-02T16:00:04+00:00
url: /2014/04/02/pushing-desired-state-configuration-to-an-azure-vm/
categories:
  - PowerShell DSC
  - Azure
tags:
  - Azure
  - PowerShell DSC

---
I had described in an earlier article that [Desired State Configuration requires WinRM listeners][1] for pushing the configuration to target systems. By default, WinRM is configured to listen on ports 5985 (HTTP) and 5986 (HTTPS). When deploying Windows VMs on Azure, you will find that the default WinRM listener is SSL (HTTPS) based and has a random public port number assigned to it. We use the Cloud Service DNS name along with the random port number assigned to the WinRM HTTPS listener.

For example, here is one of the Azure VMs I created. This is running Windows Server 2012 R2 and has the WinRM HTTPS listener created.

![](/images/azurevmdsc.png)

This is it for now. In a later article, I will show how to deploy HTTP listeners and use them for pushing DSC configuration.


[1]: /2014/04/01/desired-state-configuration-and-the-remoting-myth/