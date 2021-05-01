---
title: Announcing PowerShell module for managing Kemp LoadMaster appliances
author: Ravikanth C
type: post
date: 2014-03-18T15:37:18+00:00
url: /2014/03/18/announcing-powershell-module-for-managing-kemp-loadmaster-appliances/
categories:
  - News
  - Kemp
tags:
  - Kemp
  - News
---
In an earlier article, I talked about [managing Kemp LoadMaster appliances][1] (both physical and virtual) using the Kemp&#8217;s RESTful API. Using RESTful API is complex because you need to know all the details of how API is implemented and what are different features available to you. Even if you figure out how to use RESTful APIs, any changes to the product or RESTful API might break your scripts.

The folks at Kemp Technologies understood this and started working on a PowerShell module for managing Kemp LoadMaster appliances. A beta version of this [module is available for download][2]. Note that you need to register to see the PowerShell module download. Using this module, you can manage both physical and virtual LoadMaster appliances.

The module is named Kemp.LoadBalancer.Powershell and there are 92 cmdlets in the beta version.

![](/images/kemp.png)

This module comes with built-in help content. So, you can use your PowerShell-Fu to explore the cmdlets, see examples, and so on.

![](/images/kemp2.png)

I will be writing more about this module and how you can use it to automate the management of Kemp LoadMaster appliances in the upcoming posts. If you want a Kemp VLM to [experiment with this module][2], go ahead and [download the trial][3]. You get a 30-day license.

You can leave your feedback or ask questions in Kemp&#8217;s [product forum][4] and the team will be more than happy to help you.

[1]: /2014/01/16/managing-kemp-loadmaster-using-rest-api-and-powershell/
[2]: http://forums.kemptechnologies.com/index.php?p=/categories/powershell-for-loadmaster
[3]: http://kemptechnologies.com/in/loadmaster-family-virtual-server-load-balancers-application-delivery-controllers
[4]: http://forums.kemptechnologies.com/