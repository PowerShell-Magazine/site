---
title: Why did I start learning PowerShell
author: Adam Bacon
type: post
date: 2012-01-16T19:00:21+00:00
url: /2012/01/16/why-did-i-start-learning-powershell/
views:
  - 16220
post_views_count:
  - 1791
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
Hi, my name is Adam and I wrote this article to try and entice other people into learning something new—and for me that was Microsoft PowerShell.

It started back a couple of years ago, but as a non-programmer, and more of a point and click person I knew that this was not going to be something I could learn in 24 hours, probably not even 48 hours. So, I began by purchasing a hardback PowerShell book &#8220;[Windows PowerShell Step By Step][1]&#8221; it seemed a good place to start, and it was. I was soon learning the way to script in PowerShell. I was soon searching on the web for everything PowerShell-related. It really was an information overload at first, thinking I would never really grasp this language, without copying examples from a book. So I purchased a few more hardback copies of PowerShell books: [Pro Windows PowerShell][2], [PowerShell in Practice][3], and [PowerShell in Action][4].

Even though my day job of being a 2nd line support technician had no requirements for scripting, one fine day a big opportunity presented itself.  I was asked to go and shadow a consultant on surveying a site for a whole new system to be implemented. Part of the survey process was to get certain information from each PC/laptop onsite that the customer already had. The way it was being done was to get on your hands and knees, record the tiny half-scratched off serial number on the PC, or going into the BIOS to retrieve this info, then boot up the PC/laptop, and record how much RAM and hard disk space each PC/laptop had, then similar but more audit checks for servers. After shadowing this consultant for one day seeing how much time and effort needed to go into this process, I went home that evening and decided it was time to unleash the PowerShell I had been learning in my own time. That was a long but re-warding evening; I had developed a script that would obtain all the information required but AUTOMATICALLY without human intervention. On bigger sites it was taking 4 consultants several days to survey one site. PowerShell on the other hand could audit me 500 computers in 10 minutes. Yes, that is correct, 500 machines audited in 10 minutes, as opposed to days, and obviously no human error in recording the details. I did make several modifications to the script when requested. I have to say that I reached to Twitter for some queries, as Twitter has some really, really cool PowerShell ‘heads’ on there who always seem more than happy to answer a PowerShell question or query. In general I have found that anyone I chat to about PowerShell is more than happy to share the knowledge, which only makes the community stronger.

This then set me to start thinking what other things I could script for the company I  work for. I knew the health checks were something I did not like doing as it was a boring point and click process and took some time to do, and obviously very repetitive. So again, with the help of the mighty PowerShell and [WASP][5] (a snap-in for PowerShell), I was able to automate this whole task, and even email myself the results!

I knew another big project was coming up to remove certain software and required SQL to be uninstalled, and after using Google, I could not find any automatic way to uninstall SQL with no intervention. PowerShell with the WASP snap-in did the job again! What was even more pleasing was to hear that this was classed &#8220;impossible&#8221; to automate from other colleagues.

I have even now been asked to write a script by our development team!

I guess the point I’m trying to get across is since I have been learning PowerShell, nothing now seems &#8216;impossible&#8217; anymore. And believe me if you read enough PowerShell it sticks!

Seems PowerShell is being integrated into more and more Microsoft products, and is ultimately the future of scripting, and possibly Windows administration.

So, what are you waiting for? There are enough free PowerShell eBooks and other documentation on the Internet to wet your taste buds, so get on it and learn something new.  Only last night looked into creating a custom troubleshooting pack for Windows 7, and again the actual bit that fixes your computer is PowerShell-scripted!

I really do hope if you have not looked into PowerShell you do after reading this. Thanks for reading, and hopefully the next thing you will be reading is PowerShell-related.

Would you like to share your story? Win one of [TrainSignal’s][6] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][7]” contest.

[1]: http://www.microsoft.com/learning/en/us/book.aspx?id=10329&locale=en-us
[2]: http://www.amazon.com/Pro-Windows-PowerShell-Hristo-Deshev/dp/1590599403
[3]: http://www.manning.com/siddaway/
[4]: http://www.manning.com/payette/
[5]: http://wasp.codeplex.com/
[6]: http://www.trainsignal.com/default.aspx
[7]: ../2012/01/11/2011/12/30/2011/12/23/2011/12/19/2011/12/12/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/