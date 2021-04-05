---
title: Azure DevTest Labs Artifact for Installing PowerShell on Linux
author: Ravikanth C
type: post
date: 2016-08-22T16:00:59+00:00
url: /2016/08/22/azure-devtest-labs-artifact-for-installing-powershell-on-linux/
views:
  - 17077
post_views_count:
  - 2792
categories:
  - Azure DevTest
  - Azure
  - PowerShell Linux
tags:
  - Azure
  - Azure DevTest
  - Linux

---
**Update:** The PowerShell on Linux artifact is now available in official Azure DTL artifact repository.

[Bartek][1] and [Ben][2] have written a great introduction article on [open source PowerShell][3]. I have been experimenting with it ever since and installed it on my lab and Azure VMs. When it comes to Azure, I also use [DevTest Labs (DTL)][4] a lot for creating my test setup. Azure DTL supports customization of VMs in the lab using artifacts. Artifacts are used to deploy and configure your application after a VM is provisioned. An artifact consists of an artifact definition file and other script files that are stored in a folder in a Git repository. We can [create custom artifacts][5] and achieve what is not supported out of the box with Azure DTL. The following video provides a walk-through of creating custom DTL artifacts.

[https://channel9.msdn.com/Blogs/Windows-Azure/How-to-author-custom-artifacts/player]

I quickly [created a DTL artifact for installing PowerShell on Linux][6]. I opened a [pull request][7] to merge this artifact into the [official artifacts repository][8] and you can deploy this from the official artifact repository.

In the official artifact repository, you can find the artifact listed with Linux VM settings in DTL.

![](/images/dtlpsl1.png)

Once you select the PowerShell on Linux artifact, you will be prompted for the package URL.

![](/images/dtlpsl2.png)

You can obtain this package URL from the [PowerShell GitHub repository&#8217;s releases page][9]. On this page, you will see links to either .deb or .rpm packages. For Ubuntu, copy the link to .deb package and for CentOS, use the .rpm link.

Enter the copied URL into the Package URL input box and click _Add_ and then click _Apply_. The artifact installation will take a few minutes and you should be able to use PowerShell in the Linux VM.

![](/images/dtlpsl3.png)

[1]: https://twitter.com/bielawb
[2]: https://twitter.com/bgelens
[3]: http://www.powershellmagazine.com/2016/08/18/open-source-powershell-on-windows-linux-and-osx/
[4]: https://azure.microsoft.com/en-us/documentation/articles/devtest-lab-overview/
[5]: https://azure.microsoft.com/en-us/documentation/articles/devtest-lab-artifact-author/
[6]: https://github.com/rchaganti/azure-devtestlab/tree/master/Artifacts/linux-powershell
[7]: https://github.com/Azure/azure-devtestlab/pull/119
[8]: https://github.com/Azure/azure-devtestlab
[9]: https://github.com/PowerShell/PowerShell/releases