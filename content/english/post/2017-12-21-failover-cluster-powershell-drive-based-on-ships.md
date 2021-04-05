---
title: 'Failover Cluster PowerShell Drive based on #SHiPS'
author: Ravikanth C
type: post
date: 2017-12-21T17:00:48+00:00
url: /2017/12/21/failover-cluster-powershell-drive-based-on-ships/
views:
  - 8274
post_views_count:
  - 4039
categories:
  - Module Spotlight
  - SHiPS
tags:
  - Modules
  - SHiPS

---
Simple Hierarchy in PowerShell ([SHiPS][1]) is a module that simplifies implementing PowerShell providers. If you are new to PowerShell providers, a PowerShell provider allows any data store to be exposed like a file system as if it were a mounted drive. In other words, the data in your data store can be treated like files and directories so that a user can navigate data via _Set-Location_ (cd) and _Get-ChildItem_ (dir or ls).

I have been looking at this and experimenting with a few providers of my own. I will write more about how to approach writing a PowerShell provider using SHiPS but wanted to give you a sneak peek into the Failover Cluster PowerShell Drive ([FailoverClusterDrive][2]).

Here is the Failover Cluster PowerShell Drive in action.

![](/images/fcdrive1.gif)

This is still an experimental module. SHiPS currenly supports only get actions. So, the mounted failover cluster drive will only be read-only. There are a few more additions I am still working on in my free time and I will push another release early next year.

<div>
  <h3>
    TODO
  </h3>
* Add support for Cluster Storage as a container
* Add support for browsing cluster resource parameters as a container
* Fix support for using the module on a system with RSAT-ClusteringTools and not a cluster node.
* Add formats for better output

Stay tuned!

[1]: https://github.com/PowerShell/SHiPS
[2]: https://github.com/rchaganti/FailoverClusterDrive
[3]: https://camo.githubusercontent.com/ce719f09863b6955b3ab6b06b3e1f54926ceb62c/68747470733a2f2f692e696d6775722e636f6d2f466a5461706f472e676966