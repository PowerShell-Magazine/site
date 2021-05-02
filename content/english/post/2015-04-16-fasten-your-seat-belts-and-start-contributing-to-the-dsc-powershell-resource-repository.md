---
title: Fasten your seat belts and start contributing to the DSC PowerShell resource repository!
author: Ravikanth C
type: post
date: 2015-04-16T11:57:51+00:00
url: /2015/04/16/fasten-your-seat-belts-and-start-contributing-to-the-dsc-powershell-resource-repository/
views:
  - 11579
post_views_count:
  - 1437
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
The DSC resource development team at Microsoft released ten waves of DSC resource kit that has almost 180 resources. There are resources for System Center products, Azure Pack, Microsoft Exchange, and many more. At the same time, the PowerShell community too started writing custom DSC resources for various other products or technologies. For example, at PowerShell Magazine, we [published over 25 resources][1] to manage Azure Operational Insights, Azure Backup, Hyper-V converged switches, and so on. Like us, there are many of you who developed resources and shared with the community.

The biggest problem with these silos is that the users of these DSC resources have to look at different sources. Also, there may be issues (there will be) and fixing these issues in silos without a way to push them upstream is a major limitation. To some extent, being in an open source repository like Github helps. This is what we believed and put all our resources on Github. Now, it&#8217;s Microsoft&#8217;s turn. For an organization as big as Microsoft and with different product teams contributing to the DSC resource repository, it is not easy to just move everything to Github overnight. They need a process to publish and sustain the resource modules. At the same time, they need to enable the community to contribute as well.

We are happy to share that Microsoft has finally put the [DSC resource kit on Github][2] as individual resource modules. You can now not just use the modules in the repository but contribute as well. There are, of course, a bunch of [guidelines for contribution][3]. If you really want your custom DSC resource module be hosted in this repository, you better follow the guidelines. If not for getting into the repo, these guidelines help you build a better continuous integration story and a better way to test your modules before you release them.

With this, I am getting back to my work of making PowerShell Magazine resources compliant so that I can push them to Microsoft repository. What are you waiting for?

[1]: https://github.com/rchaganti/DSCResources
[2]: https://github.com/PowerShell
[3]: https://github.com/PowerShell/DscResources/blob/master/CONTRIBUTING.md