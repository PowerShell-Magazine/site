---
title: How I started to love PowerShell
author: Luc Dekens
type: post
date: 2012-01-11T19:00:59+00:00
url: /2012/01/11/how-i-started-to-love-powershell/
views:
  - 9486
post_views_count:
  - 1432
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
### Prologue

My first encounter with PowerShell dates from late 2006. One of my colleagues came back from Microsoft IT Forum 2006 in Barcelona and was very enthusiastic about something that was called PowerShell. It was going to be the universal language to be used for all management of the Microsoft Back Office. I installed it and immediately liked how simple it was to achieve instant results. Writing a PowerShell script was nearly as if one was writing instructions in plain English. But since a lot of our 3th party products didn’t support PowerShell yet, it sort of moved to a backburner.

### Episode 1 &#8211; 2007

Comes 2007 and a management decision to virtualize our Intel server park. We decided to go for the VMware solution. And as we were migrating physical servers to virtual servers, it became obvious that we would need some kind of automation to allow a reliable management of this new environment. I started looking at Perl, which was at that time the preferred automation language for VMware vSphere. But then by chance, I assume I was really feeling lucky in Google, I discovered a post about a new tool, called the VI Toolkit, to manage the VMware VI environment. And this VI Toolkit was a snap-in for PowerShell.

From some more Googling I learned that the Program Manager for this VI Toolkit would be presenting a session at VMware TSX 2007 in Nice. So off to France I went.

### Episode 2 &#8211; 2008

After the session, which was called “[_TA01 &#8211; Managing VMware With PowerShell_][1]”, I talked with Carter Shanklin, and was told I could get access to the closed beta of the VI Toolkit. Back home I downloaded the software and started following the VMTN Community for the VI Toolkit. It was in that Community that I first met Hal Rottenberg. Since Hal had already been playing with the VI Toolkit a bit longer, his replies to my beginner questions were extremely useful. By the time the VI Toolkit snap-in was made available as a public beta, in March 2008, I was able to manage most of VMware environment with PowerShell and the VI Toolkit.

Instead of going to another language to manage the parts that were not (yet) covered by any of the VI Toolkit Cmdlets, I used the SDK APIs. The reason that this was possible was thanks to the Get-View Cmdlet, which provided access to all the methods and properties from the SDK APIs. That way (nearly) everything was possible. I’m convinced that part of the popularity that the VI Toolkit achieved, was thanks to the presence of this Get-View Cmdlet from day 1. What is not available through a Cmdlet can be done through one of the SDK API.

I became a regular contributor to the VMTN VI Toolkit Community and I became more and more convinced that this VI Toolkit was the ideal vehicle to manage and automate a VMware environment. Also in 2008, Hal started writing his “[_Managing VMware Infrastructure with Windows PowerShell_][2]” book and I was honored to be asked as a technical editor for the book.

### Episode 3 -2009-2011

All of a sudden this VI Toolkit, which was renamed to PowerCLI end of 2010, became an important part of my life. In 2009 I presented, together with Hal, my first VMworld session. In 2010 and 2011 I presented sessions at VMworld US and EMEA together with Alan Renouf. As proof of the immense popularity of PowerCLI, we had to give several repeats of our sessions.

In 2010-2011 I was permitted to co-author a book, called the “[_VMware vSphere PowerCLI Reference_][3]”, together with 4 fantastic co- authors. Alan, Glenn, Jonathan and Arnim, thanks for the ride guys. But to be honest, there are 2 events that I will most probably tell my grandchildren. During our VMworld in 2010, we had the father of PowerShell, Jeffery Snover, sitting in the audience.  The other event was my invitation to attend the first PowerShell Deep Dive in 2011. I have books from more than half of the people present in the session’s room.

### Why PowerCLI and PowerShell?

I get this question asked a lot. I have a few reasons that were important for me. PowerShell is defined/built in such a way that a 1st-day user can be productive within minutes. That fact that the verb-noun constructs nearly read as plain English helps a lot. The language comes with a number of Cmdlets that allow you to do stuff that would otherwise take hours of coding. The Sort-Object and Group-Object Cmdlets are some of my all-time favorites. What is not available as a Cmdlet is most probably available as a method in the .Net framework. The fact that PowerShell gives you hassle-free access to this huge environment extends the possibilities of scripts enormously.

### The future

From VMware I see more and more functionality being added to their current PowerCLI offering. At the last VMworld they announced for example a complete set of Cmdlets to interact with vCD, their cloud management product. With Windows 8 Server and PowerShell v3 around the corner, I ultimately envisage all my servers to be core only and completely managed from PowerShell scripts. For someone who loves working/playing with PowerShell, the future looks bright.

Would you like to share your story? Win one of [TrainSignal’s][4] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][5]” contest.

[1]: http://vimeo.com/3352921
[2]: http://halr9000.com/article/716
[3]: http://www.powerclibook.com/
[4]: http://www.trainsignal.com/default.aspx
[5]: ../2011/12/30/2011/12/23/2011/12/19/2011/12/12/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/