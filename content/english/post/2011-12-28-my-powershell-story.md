---
title: My PowerShell story
author: Thiyagarajan Parthiban
type: post
date: 2011-12-28T19:00:32+00:00
url: /2011/12/28/my-powershell-story/
views:
  - 7035
post_views_count:
  - 1071
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
I used to be VBScripter, but not anymore. As I learned PowerShell, I slowly stopped using VBScript and now, I rarely write VB scripts. It&#8217;s been more than 5 years since PowerShell was released and I still learn PowerShell every day.

I would like to share my experience, resources, some tips and hopefully some new beginners will learn from my mistakes.

### Pipeline

As a VB scripter, I started approaching PowerShell as a scripting language. I would write a small script to solve something and later realized, I could have done the same thing in one line using the pipeline, so don’t miss that one.

### Get-Help

Read the PowerShell Help! Period. Start reading those errors which show up in **red.** Unless you read those error messages and try to understand them, you won’t learn much. Making mistakes and trying out different things, you’ll get errors, that is good. Read and understand them, ask why and soon you can explain how PowerShell does things the way it does. I have learned few things the hard way, so it will really help you a lot by reading this. You can probably download this offline PowerShell CHM file from [here][1].

### Hey Scripting Guy Blog

Every day Ed Wilson, The Scripting Guy, writes a blog about PowerShell , 365 days a  year. You can search through the archives for older posts to get started on PowerShell and learn different techniques. You can visit the blog [here][2]. If you still want to have fun with it, every year Ed conducts [the Scripting Games][3] , participate and put your skills to work. You will get a chance to write real world scripts and also get a chance to see how other people scripts.

### Powershell.com

I like the [tips][4] which are posted here on a daily basis; they are really useful and you can search through the archives for previous tips as well. They even host a monthly webcast which are really useful, you just have to sign up and you can attend for free.

### Powershell Communities

Look for a PowerShell community near your location, check out [www.PowershellGroup.org][5]. This website has most of the PowerShell communities around the world, try to join whichever is near you. These communities also conduct offline meetings in some place where you can attend in person and meet new people who are trying to learn PowerShell.

### Tools

I like how PowerShell emits Objects, unlike VBScript where I had to do a lot of string manipulation to get what I want. I instantly liked the [PowerGUI][6] Editor for writing my PowerShell scripts, the only reason I like it the most is because of the IntelliSense feature, so you can explore PowerShell better. For Example, if I type Get- ,PowerGUI will automatically show me the commands that start with the Get verb, this also applies to object properties:

![](/images/myStory1.png)

Here are some more resources, which might be helpful:

  1. List of [free e-books][7], by Jason [Five part webcast series][8] on PowerShell by Ed Wilson
  2. If you need help on your PowerShell problem, there are people to help you on the TechNet forums. 
      1. [Powershell Forum][9]
      2. [Scripting Guys Forum][10]

I feel that all the efforts you invest won’t go to waste, now that we see that most of Microsoft Products are tightly integrated with PowerShell. Thanks to the PowerShell Magazine for giving me a chance to write about my PowerShell story.

Would you like to share your story? Win one of [TrainSignal’s][11] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][12]” contest.

[1]: http://blogs.msdn.com/b/powershell/archive/2011/05/17/download-the-updated-core-help-chm.aspx
[2]: http://blogs.technet.com/b/heyscriptingguy/
[3]: http://blogs.technet.com/b/heyscriptingguy/archive/2011/02/19/2011-scripting-games-all-links-on-one-page.aspx
[4]: http://powershell.com/cs/blogs/tips/
[5]: http://www.powershellgroup.org/
[6]: http://www.powergui.org/index.jspa
[7]: http://www.hofferle.com/archives/624
[8]: http://technet.microsoft.com/en-us/scriptcenter/dd742419
[9]: http://social.technet.microsoft.com/Forums/en/winserverpowershell/threads
[10]: http://social.technet.microsoft.com/Forums/en-US/ITCG/threads
[11]: http://www.trainsignal.com/default.aspx
[12]: ../2011/12/23/2011/12/19/2011/12/12/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/