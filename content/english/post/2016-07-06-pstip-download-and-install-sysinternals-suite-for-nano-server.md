---
title: '#PSTip Download and install Sysinternals Suite for Nano Server'
author: Jan Egil Ring
type: post
date: 2016-07-06T16:00:27+00:00
url: /2016/07/06/pstip-download-and-install-sysinternals-suite-for-nano-server/
views:
  - 14430
post_views_count:
  - 3949
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The Sysinternals Suite, which contains many tools an IT Professional should have in the tool belt, is now available for Nano Server. Traditionally the tools have been packaged as 32-bit applications, which automatically extract and run the 64-bit version when run on a 64-bit system. Since Nano Server is 64-bit only, the tools had to be rewritten

to work on Nano Server. Naturally, all tools are not ported, such as those with a graphical user interface (GUI) (e.g. Process Monitor). However, some of the GUI only tools do work remotely against Nano Server. You can get more details and watch demos in [this][1] Channel 9 interview with Andrew Mason from the Nano Server team and Mark Russinovich, the creator of the Sysinternals tools.

The new Sysinternals Suite for Nano Server is available as a separate download on the [Sysinternals Suite on TechNet][2]:

![](/images/nano1.png)

Installing and extracting the file on Nano Server is not trivial to do locally, since Invoke-WebRequest is not available on Nano Server and the Expand-Archive cmdlet does not work on Nano in Windows Server 2016 Technical Preview 5 (will work in RTM).

However, that does not stop us from automating installation process. The two cmdlets can simply be run on the local machine and the extracted files can be copied to Nano Server using the new remote file copy capabilities [introduced][3] in PowerShell 5.0:

Copy-Item now lets you copy files or folders from one Windows PowerShell session to another, meaning that you can copy files to sessions that are connected to remote computers, (including computers that are running [Windows Nano Server][4], and thus have no other interface). To copy files, specify PSSession IDs as the value of the new -FromSession and -ToSession parameters, and add –Path and –Destination to specify origin path and destination, respectively. For example, Copy-Item -Path c:\myFile.txt -ToSession $s -Destination d:\destinationFolder.

A PowerShell script demonstrating this technique is available in [this Gist][5] on GitHub.

The following tools are included in the initial release:

![](/images/nano2.png)

Example usage of one of the tools &#8211; PsInfo:

![](/images/nano3.png)

[1]: https://channel9.msdn.com/Shows/Defrag-Tools/Defrag-Tools-164-Sysinternals-Nano
[2]: https://technet.microsoft.com/en-us/sysinternals/bb842062
[3]: https://msdn.microsoft.com/en-us/powershell/scripting/whats-new/what-s-new-in-windows-powershell-50
[4]: http://blogs.technet.com/b/windowsserver/archive/2015/04/08/microsoft-announces-nano-server-for-modern-apps-and-cloud.aspx
[5]: https://gist.github.com/janegilring/48ec8c32ba4972e08a0f1470400f29ea