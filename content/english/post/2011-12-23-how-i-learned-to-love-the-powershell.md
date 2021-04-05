---
title: How I learned to Love the PowerShell
author: Clint Bergman
type: post
date: 2011-12-23T19:00:52+00:00
url: /2011/12/23/how-i-learned-to-love-the-powershell/
views:
  - 5431
post_views_count:
  - 1031
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
When I first discovered the existence of PowerShell it was in its pre-release stages and still known as Monad.  At first I couldn’t believe what I was seeing and hearing, “A Windows shell that uses objects?  Awesome!  Look at all the cool stuff they’re doing with it!” I exclaimed to no one in particular.  Having only used cmd.exe and DOS shells of old, and with some small amount of experience with VBScript and C/C++, the thought of having an object-oriented interactive shell was very exciting.  I quickly downloaded the Monad beta and got it installed.  Within a few moments I was greeted with the classic PowerShell prompt:

`PS C:\>`

Then I realized&#8211;I had no idea what to do next.  Installing PowerShell was easy; figuring out how to use it to do what I wanted would prove significantly more difficult.  Eventually I learned the first of what I think of as the three “pillar cmdlets” of PowerShell: Get-Help.  Not long after that I learned of the second: Get-Command, and later the third: Get-Member.  To me, and in my experience to most of the people I have shared PowerShell with, the most difficult part of learning to use and love PowerShell is learning how to use PowerShell to explore itself.  It’s a bit like having to pick yourself up by your own bootstraps.

At the beginning of my learning curve I would force myself, time permitting, to figure out how to accomplish a task using PowerShell.  Often this meant typing “Get-Command” at the prompt and looking for a cmdlet that seemed promising.  This was a challenge of will because usually I could accomplish the same task in a few minutes using methods I was accustomed to.  There was much trial and error on my part at that time; proper documentation and community knowledge were still in their infancies.  Once _PowerShell in Action_ was published, I read it cover to cover even though there were whole chapters I wouldn’t understand until much later.  I’m looking at you extensible type system.

Why go to all this trouble to learn some new technology?  Having the interactive shell made solving a challenge in a repeatable manner much simpler.  I could rapidly discover syntax problems at the command prompt instead of having WScript blow up in my face because I can never get LDAP filter syntax right the first time.  I suppose I invested all the time I did in learning PowerShell because I believed that, as far as Windows went, it was better and easier to automate tasks with than anything else I had access to.  Now that a PowerShell scripting interface has been mandated by the common engineering criteria at Microsoft, I’m very glad that I chose to invest time in learning it.

Today there is a vast community that has grown up around PowerShell that is, in my experience, happy to welcome and help the newcomer.  I believe it is the best resource any newcomer has at their disposal.  Nowhere have I experienced that help more than in the official PowerShell forums.  I’m certainly grateful to have had so many people willing to lend a hand to me as I learned, and today I try to give back to other newcomers through the forums, blogging, and training workmates.  For any newcomer today I have only a few bits of advice:  Remember Get-Help, Get-Command, and Get-Member.  Ask questions of the community.  Do your best to give back when possible.  Enjoy the journey!  I don’t believe a Windows admin could better invest time in learning a different technology.

Would you like to share your story? Win one of [TrainSignal’s](http://www.trainsignal.com/default.aspx) PowerShell training courses? Learn more about the [How I Learned to Stop Worrying and Love Windows PowerShell](/2011/12/19/2011/12/12/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/ ) contest.
