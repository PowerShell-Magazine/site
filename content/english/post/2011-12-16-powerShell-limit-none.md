---
title: PowerShell -Limit none
author: Joel Reed
type: post
date: 2011-12-16T19:00:06+00:00
url: /2011/12/16/powerShell-limit-none
views:
  - 8596
post_views_count:
  - 1495
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
I have been employed as a Windows systems administrator for the last 14 years, cutting my enterprise teeth back in the NT days. I’ve worked for governments, dot coms, and non-tech oriented corporations. I’ve always been “The Windows Guy”. I started off supporting a handful (and ever increasing number) of Windows servers in a Novell environment. Later I found myself “The Windows Guy” again in a mostly Linux dot com shop. Currently I am with a large financial company that is mostly Windows but still Java at heart, so my .NET corner is once again the minority.

I do not recall my first script in PowerShell. It probably was not some miraculous piece of automation, saved my employer millions, or won any business deals. If I was able to find it in a backup I would probably be horrified, disavowing knowledge of it. I am sure I eagerly emblazoned my name and email in the top comments, ‘sigh’. Being a heavy user of VBscript, before PowerShell, at some point I realized I was authoring scripts that were effectively “VBScript” in PowerShell. I eventually used VBScript less and less, and through many wonderful and insightful posts from the PowerShell blogosphere I transitioned and really began to use and understand PowerShell for PowerShell.

I think sometimes we can look at PowerShell and our corners of the Microsoft world and say “I don’t really have anything that I can use with PowerShell”. You may not run this or that piece of the Microsoft stack, or even the latest and greatest. Many of us are just beginning to use v2 completely in our enterprises with transitions to Server 2008 R2 and Windows 7. There will be a tremendous amount of awesomeness in PowerShell v3 and Windows 8; however many of us will continue to use PowerShell v2 for a long time. Never get fooled into thinking you have to have “a better environment” to get the most out of PowerShell. Or limit yourself because there is no PowerShell module or cmdlets for what you administer in your enterprise, be it legacy Windows, custom applications and services, or entirely non-Windows systems.

Here are some tips from my endeavors over the years. Unfortunately they are not fully detailed but enough for you and your favorite search engine to build on.

&#8211; Quit using cmd.exe. Running net, ipconfig, netsh, ping, or netstat works just fine in PowerShell, as do most other “native” commands. Perform the exercise of finding the PowerShell way of accomplishing the same task, i.e. Get-Service. If there is no single cmdlet that mirrors the command write a function that gets you there.

&#8211; The type accelerator is very handy. For example:

```powershell
$content = Get-Content -Path SomeFile.xml
```

This is a first class object. Many configuration files, even non Microsoft, are XML based.

&#8211; PowerShell’s .NET integration, a more advanced path, has capabilities that can be harnessed today. The MSDN .NET documentation is great and many entries have PowerShell examples in the comments. Classes like System.Net.WebClient and System.Net.Sockets can be used against web services and Linux daemons.

&#8211; For those of us with legacy systems and applications, the COM object can still be used under PowerShell.  Check the help for New-Object, load up an instance, and dig in with Get-Member. You will be amazed at what is possible.

&#8211; Regardless of these or other methods used to solve problems, always output objects. If you live by this rule your future self with thank you.

&#8211; Find and attend a PowerShell user group, they are popping up everywhere. I attend <a href="http://www.azposh.com/" target="_blank">AZPosh</a> which also provides a live interactive broadcast of meetings. This is great if there are no groups in your area.

At the end of the day the best PowerShell scripts you author will be the ones that solve your unique problems. Whether that is simplifying your daily administration or satisfying your enterprises business need.

Joel Reed.

Would you like to share your story? Win one of [TrainSignal’s][1] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][2]” contest.

[1]: http://www.trainsignal.com/default.aspx
[2]: ../2011/12/09/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/