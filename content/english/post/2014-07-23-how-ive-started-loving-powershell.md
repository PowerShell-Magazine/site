---
title: How I’ve started loving PowerShell
author: Fabien Dibot
type: post
date: 2014-07-23T16:00:57+00:00
url: /2014/07/23/how-ive-started-loving-powershell/
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
So, it’s my turn to present myself!

I’m Fabien, 32 years old and I started using PowerShell in 2008 because I’m a lazy guy. I&#8217;ve always liked to build things in order to make them work. I think you need automation to handle your daily workload, and the day a colleague showed me the Exchange reports graphics he&#8217;s doing I felt in love with this crazy command line and the simplicity of achieving complex tasks.

So I decided to use PowerShell to enhance the automation of our software deployment with its database for our customers. I won&#8217;t say everything went well at start, I’ve spent many times googling and finally found an amazing French PowerShell community site (<a href="http://powershell-scripting.com/" target="_blank">Powershell-Scripting.com</a>)! During this learning phase, I’ve felt in love with three cmdlets: _Get-Member_, _New-Object,_ and _ForEach-Object_! As everything is an object in PowerShell nothing can be more helpful.

This community was led by [Arnaud Petitjean][1] and [Laurent Dardenne][2]. They’ve been working with PowerShell for ages and helped me kindly answering my beginner’s questions. With cmdlets and .NET assemblies, I could achieve my goals and as I spent my time on forum I started to contribute to help the community. I’m usually focused on Microsoft technologies, however, I had to work on upgrading a VMware ESX farm in another job. I already knew that PowerCLI existed, but I couldn&#8217;t imagine how your life can be so good with these two products combined. Imagine the happiness of upgrading a complete infrastructure with just scripts. Thanks to the “fearsome” _Get-View_ cmdlet, you have access to all SDK methods. Automation is the key for success in ALL infrastructures. So I started a blog to share my findings, scripts and gave some tips to help people adopt PowerShell.

My next task offered me the opportunity to go deep inside PowerShell 4.0, and I have to admit this PowerShell version coupled with Windows Server 2012 cmdlets was pure happiness to work with. New cmdlets allowed me to configure teaming on physical servers with _New-NetLbfoTeam_ and configure all iSCSi initiators, a serious gain of time. Another awesome cmdlet to work with third party software is Invoke-RestMethod&#8211;perfect match to handle web services!

But one thing bothered me&#8211;in France, nobody knows about PowerShell and how easy is to use it, doing awesome stuff avoiding hours of coding. After seeing PowerShell sessions at TechDays, I felt that we need to inform the French IT community about the capabilities of PowerShell.

During TechDays 2014, [Pascal Sauliere][3] gave me the opportunity to present two sessions with [Carlo Mancini][4]&#8212;[one about remoting][5], and [the other one][5] for beginners about tips & tricks. I was amazed to see that attendees didn&#8217;t know about PowerShell remoting before this session. But, they loved it after they learned about it!

Currently, I manage Oracle, MySQL and SQL Server as well as Windows Server only with PowerShell. Sometimes it looks like we don&#8217;t have any limits&#8211;when something is not designed as a cmdlet, you can directly use .NET Framework to help you find a solution!

Now, as Microsoft is taking care of supporting PowerShell in every server product and in parallel improves the PowerShell core functionalities, PowerShell will be a feature that every IT guy will have to use daily. For example, Desired State Configuration opens a door to a new world of automation, and I think the future for PowerShell users will be awesome!

Let me share a few sweet short code snippets I use on a daily basis:

```
#Generate a GUID and export it to clipboard
[GUID]::NewGuid().ToString() | clip

# Test if account is "Administrator"
[bool]((whoami /all) -match "S-1-16-12288")

# Validate an IP address
if ($ip -as [IPAddress]) {"OK"}
```

Have you ever imagined a Linux IT guy without the shell skills? It&#8217;s the same for Windows IT guys now.

[1]: http://mvp.microsoft.com/en-us/spotlight/Arnaud%20Petitjean-20130723121339
[2]: http://laurent-dardenne.developpez.com/
[3]: https://twitter.com/psauliere
[4]: https://twitter.com/sysadm2010
[5]: http://www.microsoft.com/france/mstechdays/programmes/2014/fiche-session.aspx?ID=b385dab1-86b6-45a2-a9b9-abcc66a2adef