---
title: 'Hyper-V Server PowerShell Drive based on #SHiPS'
author: Ravikanth C
type: post
date: 2018-01-02T17:00:29+00:00
url: /2018/01/02/hyper-v-server-powershell-drive-based-on-ships/
views:
  - 9832
post_views_count:
  - 5357
categories:
  - Module Spotlight
  - SHiPS
tags:
  - Modules
  - SHiPS

---
In an earlier article, I had written about a [PowerShell provider for Failover Clusters][1] written using the SHiPS provider framework. I have been experimenting with this a bit and made a few more providers.

In today&#8217;s article, I am introducing the [Hyper-V Server PowerShell provider][2].

Using this provider, you can connect to local and remote Hyper-V hosts and browse the virtual machines and virtual networks as if they are folders on a file system.

![](/images/HyperVDrive.gif)

Once again, like every other provider I am writing, this is experimental as well. I am writing these to understand what needs to be considered as part of the provider design and implementation. So, the final version of these providers may look and function differently.

<div>
  <h3>
    TODO:
  </h3>

  <div>
    &#8211; Add support for Hyper-V Host properties as leaf
  </div>

  <div>
    &#8211; Fix support for using the module on a system with RSAT-ClusteringTools and not a Hyper-V host.
  </div>

  <div>
    &#8211; Add formats for better output
  </div>
</div>

Follow the [Github repository][2] for information on any future updates.

[1]: http://www.powershellmagazine.com/2017/12/21/failover-cluster-powershell-drive-based-on-ships/
[2]: https://github.com/rchaganti/HyperVDrive