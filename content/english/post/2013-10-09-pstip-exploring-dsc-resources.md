---
title: '#PSTip Exploring DSC resources'
author: Ravikanth C
type: post
date: 2013-10-09T18:00:26+00:00
url: /2013/10/09/pstip-exploring-dsc-resources/
categories:
  - PowerShell DSC
  - Tips and Tricks
tags:
  - Tips and Tricks
  - PowerShell DSC

---
**Note**: This tip requires PowerShell 4.0 or above.

In Windows Server 2012 R2 RTM build, Microsoft added a cmdlet called _Get-DSCResource_. This cmdlet helps us explore the DSC resources available on the local computer.

![](/images/dsc-resource.png)

The _ImplementedAs_ property of the DSC resource tells us whether it is a PowerShell Script module or a Binary module.

![](/images/implementedas.png)

The “_Properties”_ property of a DSC resource tell us what attributes can be used as a part of the configuration document.

![](/images/properties.png)

If we don’t know how to use the attributes of a DSC resource in a configuration document, we can use the _–Syntax_ switch parameter of the _Get-DSCResource_ cmdlet.

![](/images/syntax.png)