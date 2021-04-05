---
title: ScriptWarrior's First PowerShell Experiences
author: Jeff Truman
type: post
date: 2011-12-05T20:00:39+00:00
url: /2011/12/05/scriptwarriors-first-powershell-experiences/
aktt_tweeted:
  - 1
post_views_count:
  - 1118
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
I was asked by a PowerShell Rock Star, [Shay Levy][1], to forward on my very first experiences with PowerShell.  Why I started to use it in the first place and what resources I employed to solve my first scripting problems.

It all started one fateful night about 3 years ago, I had been supporting a major software release for the company I was working for at the time.  The company that I was then employed by wrote Social Networks for large organizations.  One such client in particular came to us and asked that we produce an entire social networking site for one of their small business credit card offerings.  My company jumped at the chance to provide such an important service to a multinational, financial institution.  The code was written and the site was deployed.  Now needless to say since I was on the server administration team for my company, it was put very black and white that the SLA for this website COULD NOT BE VIOLATED.  If the client at any time felt we were not living up to expectations, they could take all the code we had written and say have a nice day. As I said, the site was deployed and all worked as expected until&#8230; the one thing that should not be an issue, became a major issue. People started to use the site!

As traffic increased an interesting scenario started to arise.  A random series of events was causing the WWW service on the network load balanced web farm servers to spin off into oblivion.  This meant that any request from the load balancer to that web server while it was in this condition would result in a broken page and generally a 404 error or a blank webpage. A blank page or a 404 error needless to say was not part of the SLA for this huge client.  These issues needed to be fixed quickly.  As these errors started to happen, my admin team went about attempting to find out why the issue was occurring in the first place.  We found some artifacts and forwarded them back to the development team to work on a fix.  In the meantime, we developed a process to mitigate the error state as quickly as possible.  Here were the steps involved:

  1. (If offsite or off hours) VPN into the office, then VPN into the production network in Orlando.
  2. Once connected to the Prod Network, log in to F5 load balancer.
  3. Find the offending server in the active F5 resource pool.
  4. Mark the NODE down so that all connections would bleed off  the offending server and would not allow new request from the NLB to be directed to the problematic server.
  5. Once all connections bled to 0. We simply reset IIS on the box.
  6. Once IIS was reset, we ran a quick simple test, and then place the box back in the resource pool.

Now what I didn't mention before is that 90% of the time these boxes were exhibiting this behavior was during the middle of the night. This went on for about 3 weeks and then I figured there must be something else that can be done.  So, one night after bouncing one of the servers, I couldn't get back to sleep.  Since I was awake anyway, I logged into the F5 development site.  I saw a new article about something called an [F5 PowerShell snap-in][2].

This intrigued me.  I had heard of PowerShell a little bit.  I knew it was much more C#-based than the horrible VBScripting I had been doing. I started to look at the documentation for the snap-in and found a script that had the building blocks of what I needed to do to automate the whole process of recovering from the error.  It truly took me about a week of pouring over the documentation and finding sample scripts before i had something even close to working.  Then one very early morning I had to get up again to fix the error on a box and I knew it was time to finish the script.  It had been nearly 2 full weeks without a full night’s sleep, and it was getting painful.  I finished the script that day and wouldn't you know it the condition

arose again the next day.  Perfect test time and it worked like a charm.

In conclusion, I tell people even now when they ask me about scripting, that you will do it through the GUI until it hurts.  Then you will script it.  There has to be a reason to script.  I don&#8217;t

write scripts just to script, there has to be a problem to solve. GO FIND A PROBLEM AND SOLVE IT WITH POWERSHELL :o)

&#8211;@ScriptWarrior

Would you like to share your story? Win one of [TrainSignal&#8217;s][3] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][4]” contest.

[1]: http://powershay.com/
[2]: http://www.f5.com/solutions/applications/microsoft/powershell/
[3]: http://www.trainsignal.com/default.aspx
[4]: http://powershellmagazine.com/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/