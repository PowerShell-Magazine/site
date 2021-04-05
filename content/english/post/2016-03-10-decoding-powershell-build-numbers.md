---
title: Decoding PowerShell build numbers
author: Jan Egil Ring
type: post
date: 2016-03-10T17:09:11+00:00
url: /2016/03/10/decoding-powershell-build-numbers/
views:
  - 21587
post_views_count:
  - 6777
categories:
  - How to
tags:
  - How to

---
There are several ways to determine what version of Windows PowerShell you are running on a computer. The most common technique is to use the $PSVersionTable automatic variable:

![](/images/psbuild1.png)

Here, we can see the PowerShell version number in the PSVersion property.

Until PowerShell 5.0 was released as part of Windows Management Framework 5.0, the build number was rounded to .0, such as 1.0, 2.0, 3.0, and 4.0. However, in this new and fast moving pace Microsoft have gotten into, we will see more frequent updates to PowerShell than before. Due to this, the PowerShell version number isn&#8217;t simply 5.0 anymore as you might have expected. Now it&#8217;s a full build number, such as 5.0.10586.117 for the [RTM release][1] of Windows Management Framework 5 for downlevel operating systems.

If you are used to testing preview build of the Windows Management Framework in earlier versions, you might be familiar with the notion of build numbers. However, it&#8217;s getting harder to keep track of what build number maps to what version of PowerShell. For example, is 5.0.10586.51 a preview version or an RTM version? Without any form of mapping information, it&#8217;s very difficult to know.

Because of this difficulty, I decided to create PowerShell function which can read a mapping table and convert the PSVersion property to a “PSFriendlyVersionName” which makes more sense to a user. Here you can see Get-PSVersion in action:

![](/images/psbuild2.png)

![](/images/psbuild3.png)

![](/images/psbuild4.png)

![](/images/psbuild5.png)

The function is reading a JSON file where the mapping information is defined:

![](/images/psbuild6.png)

Since Microsoft does not provide any PowerShell build number reference, it is up to us – the community – to maintain this mapping information. I have added the RTM build numbers down to version 2 (it does not make sense to add version 1, since PowerShell Remoting which the function leverages was introduced in version 2), but there are a lot of build numbers for preview versions to add.

I have put both the Get-PSVersion function and the JSON file in a PowerShell module, in order to make it easy to download from the PowerShell Gallery. This means you can simply type **Install-Module -Name PSVersion**  to install it directly from the gallery. Or, you can of course use Save-Module first if you want to inspect the content before installing it. The module is also [available][2] on my GitHub account, where you can fork it and send a pull request if you have any contributions to either the Get-PSVersion function or the JSON mapping file. I have also created a TechNet Wiki site called [Windows PowerShell build numbers][3] as a reference for those who don\`t want to leverage the PSVersion module.

[1]: https://www.microsoft.com/en-us/download/details.aspx?id=50395
[2]: https://github.com/janegilring/PSVersion
[3]: http://social.technet.microsoft.com/wiki/contents/articles/33573.windows-powershell-build-numbers.aspx