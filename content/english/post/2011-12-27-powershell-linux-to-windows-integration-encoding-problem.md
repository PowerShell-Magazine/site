---
title: PowerShell Linux to Windows integration (encoding problem)
author: Dmitri Guslinsky
type: post
date: 2011-12-27T19:00:36+00:00
url: /2011/12/27/powershell-linux-to-windows-integration-encoding-problem/
views:
  - 23978
post_views_count:
  - 1948
categories:
  - Debugging
tags:
  - Debugging

---
In my life I try to be on the edge of IT technology and if I find something interesting I try to be in touch with it. I&#8217;m a big fan of UNIX based systems and impressed by their reliability and flexibility. That is why I love Linux and Mac OS.

I realize that Windows is the third major system, it’s still alive and used in most of companies around the world. Windows 7 looks better protected and much more reliable. So, I studied it  and looked the new technologies it offered. The weak points in Windows were the reliability, security, virus protection and command line interface. I&#8217;m a big enthusiast of CLI (command line interface) on Linux systems and this article is dedicated to that.

The first time I heard about Windows PowerShell I said &#8220;Wow, that is crazy&#8221;. What a great idea, make the command line shell in Windows really powerful. I won&#8217;t compare it to UNIX bash here. It is clear that PowerShell gives system administrators the ability to automate their job and a way to control the Windows systems through slow network connections. That is great.

In addition to my CLI interests, I&#8217;m a big enthusiast of OS integration. I’ll explain what I mean. Usually in companies I’ve been at, there are offices with Windows 7 desktop computers and laptops. Some have Windows Servers or Linux. The companies who do design, advertisement, computer graphics, post-production or TV have all three systems in their network. That means: Mac OS, Linux, Windows 7 and even Windows XP.  Control and integration is a problem. I&#8217;m looking for technology which can help. PowerShell and SSH  may make this happen.

Until now, Windows 7 didn’t have SSH  out of the box. It uses an outdated \`telnet&#8217; for console connections. Thanks to [@ShayLevy][1] for the tip on the PowerShell SSH Server from [/n Software][2]. It looks great for these types of integration. Plus, they have a free version with a one connection limit.

The problem I faced &#8211; we have a Russian version of Windows 7. That means if I try to connect it with SSH  from Linux I get a question mark in PowerShell replies.

![](/images/debug1.png)

The problem is different locales. The Linux box has modern UTF-8, the Windows have their own WIN-1251 for the Russian language. So, PowerShell Server from /n Software translates the reply incorrectly encoded.

I solved this by tweaking the registry for PowerShell Server.

![](/images/debug2.png)

First, I need to make a new string parameter in \`HKEY\_LOCAL\_MACHINE ➜ SOFTWARE ➜ PowerShellInside  ➜ PowerShellServer&#8217;. The \`WireEncoding&#8217; should be set to \`UTF-8&#8242;.

![](/images/debug3.png)

Then I run PowerShell server as a Windows Service, restart it, and connect through SSH again. It now works with the correct Russian responses.

![](/images/debug4.png)

Happy Holidays 🙂

[1]: http://twitter.com/shaylevy
[2]: http://www.powershellinside.com/powershell/