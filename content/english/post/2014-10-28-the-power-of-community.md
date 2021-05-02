---
title: The Power of Community
author: Jan Egil Ring
type: post
date: 2014-10-28T15:00:45+00:00
url: /2014/10/28/the-power-of-community/
categories:
  - Community
tags:
  - Community
---
Since the early days of Windows PowerShell, the community around the product has grown to a large and active user base. In this article, we will look at how the community can influence future releases.

Windows PowerShell is software. Any software, in general, will always have bugs as well as the need for new features and improvements. Microsoft has a feedback channel called Microsoft Connect where customers can report bugs and provide suggestions for feedback to their software. There is a dedicated Connect website for Windows PowerShell: [connect.microsoft.com/PowerShell ][1]

![](/images/Microsoft_Connect.png)

In addition to submitting your own items, you can also vote for items posted by others. Here is an [example][2]:

![](/images/Microsoft_Connect_example1.png)

Another feedback opportunity you should know about Windows PowerShell is regarding the help system. Since version 3, the help system is updatable (_Update-Help_), which makes it possible for Microsoft to update the help files with more information as well as correcting errors on a regular basis. To report errors or suggestions for the help system, you can use the e-mail alias write-help (at) microsoft (dot) com.

### UserVoice

There are also other feedback channels available for PowerShell-related technologies such as Azure Automation and Service Management Automation (SMA). Microsoft Azure is using [UserVoice][3], a software-as-a-service provider of customer support tools. Here is a few categories from the Azure channel at UserVoice relevant to PowerShell:

  * [Azure Automation][4]
  * [Scripting and Command Line Tools][5]
  * [Service Management Automation][6] (part of Azure Pack)

![](/images/UserVoice.png)

[ISESteroids][7] is another product leveraging UserVoice. I recently [submitted an idea][8] for a new feature in ISESteroids regarding region expansion, and I was positively surprised when I saw the feature implemented in the latest preview only 2 days later.

### GitHub

There are also a number of Microsoft projects on GitHub. [OneGet][9]&#8211;a unified package management module to be released in PowerShell 5.0&#8211;is a great example. In the OneGet project on GitHub, we can submit [issues][10] and [pull requests][11] among other things.

![](/images/GitHub1.png)

Another example is the [Hardware Management Module][12], a PowerShell module for managing hardware out-of-band via the WS-Man protocol, which was released on GitHub a few months ago.

### Conferences

The last feedback channel to mention in this article is going to conferences where you can meet and talk directly to product team members in order to provide feedback. Microsoft TechEd (rebranded to [Microsoft Ignite][13] going forward) and the [PowerShell Summit][14] are great examples of such conferences.

[1]: connect.microsoft.com/PowerShell%20
[2]: https://connect.microsoft.com/PowerShell/feedbackdetail/view/973434/install-uninstall-pswawebapplication-should-not-use-write-host
[3]: https://www.uservoice.com/about/
[4]: http://feedback.azure.com/forums/246290-azure-automation
[5]: http://feedback.azure.com/forums/216665-scripting-and-command-line-tools
[6]: http://feedback.azure.com/forums/255259-azure-pack/category/83215-service-management-automation
[7]: http://www.powertheshell.com/isesteroids2/
[8]: https://scriptinternals.uservoice.com/forums/246488-general/suggestions/6529536-toggle-expansion-for-current-region-using-a-keyboa
[9]: https://github.com/OneGet/oneget
[10]: https://github.com/OneGet/oneget/issues
[11]: https://github.com/OneGet/oneget/pulls?q=is%3Aopen+is%3Apr
[12]: https://github.com/WindowsPowerShell/Hardware-Management-Module
[13]: http://ignite.microsoft.com/
[14]: http://powershell.org/wp/community-events/summit/