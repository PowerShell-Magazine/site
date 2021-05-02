---
title: Microsoft announced the CTP release of Windows PowerShell Desired State Configuration for Linux
author: Ravikanth C
type: post
date: 2014-05-20T02:52:52+00:00
url: /2014/05/19/microsoft-announced-the-ctp-release-of-windows-powershell-desired-state-configuration-for-linux/
categories:
  - News
  - PowerShell DSC
  - Linux
tags:
  - News
  - PowerShell DSC
  - Linux

---
When I was speaking at DevOpsDays India and [demonstrating Windows PowerShel DSC][1] (6 months ago), several people in the room had a laugh as I&#8217;d mentioned Microsoft and OpenSource. I&#8217;d mentioned that DSC is based on standards and it will, one day, be extended to Unix and Linux platforms. I spoke about configuration management of your entire data centre based on standards like CIM and WS-MAN. And, that is available today!

If you were at [TechEd or watched it online][2] or followed #msteched on Twitter, you would have noticed the buzz around an Open Source announcement by [Jeffrey Snover][3]. Microsoft released a CTP version of [DSC for Linux on GitHub][4].

### Prerequisites

- Development tools (g++, GNU make)
- OMI 1.0.8 (https://collaboration.opengroup.org/omi/documents/30532/omi-1.0.8.tar.gz)
- Windows PowerShell Desired State Configuration for Linux source (https://github.com/MSFTOSSMgmt/WPSDSCLinux/releases/download/v1.0.0-CTP/PSDSCLinux.tar.gz)
- Python 2.5 or later and python-devel

Building OMI 1.0.8 requires the following packages:

- pam-devel
- openssl-devel

This initial CTP release has only a few DSC resources available!

- nxFile – manage files and directory state
- nxScript – runs script blocks on target nodes
- nxUser – manages Linux users
- nxGroup – manages Linux groups
- nxService – manages Linux services (System-V, Upstart, SystemD)

Hey, this is open sourced. So, let us build this together!

A complete step-by-step guidance for this is available at: http://blogs.technet.com/b/privatecloud/archive/2014/05/19/powershell-dsc-for-linux-step-by-step.aspx

This is really an exciting time and I will be closely following the development and writing my observations and learning here. Watch this space.

[1]: http://devopsdays.org/events/2013-india/presentation/
[2]: http://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DCIM-B417#fbid=?hashlink=fbid
[3]: http://www.jsnover.com/
[4]: https://github.com/MSFTOSSMgmt/WPSDSCLinux