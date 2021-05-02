---
title: How braces and semicolons became a part of my life
author: Manoj Ravikumar
type: post
date: 2014-07-25T16:00:19+00:00
url: /2014/07/25/how-braces-and-semicolons-became-a-part-of-my-life/
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
A mishandled case, an irate customer and a top box (i.e. exceeded customer expectations in a post case resolution survey) at the climax. No, this isn’t a premise for a detective potboiler but a support case that was transferred to me when I was a support engineer with Microsoft Global Technology Support Center (GTSC). The customer was using a SharePoint form that the end users can fill in and this form will create an AD account for the user and add it to a specific Organizational Unit. However, to get access to the SharePoint Site, the AD user account created should be a member of a specific AD group which the current setup couldn’t do and had to be done manually. Using Quest (now Dell), AD PowerShell Cmdlets, I wrote a Windows PowerShell snippet that checks for created users and moves them using the scheduled task to a specified OU thereby granting access. More on this in the [blog post][1].

This might not be a breakthrough innovation nor is a thousand plus lines of code but what matters is that it helped the client resolve her concern without investing in third-party products. Upon case review, she gave me a top-box (9/9). My first top box via a Windows PowerShell script and I had so much fun writing it. Since then I haven’t looked back. Be it PowerShell remoting, PowerShell Workflow, Desired State Configuration, or OneGet, the automation engine continues mystifying me.

Like most of the PowerShell newbies, I started my journey with Windows PowerShell by running the Get-Service cmdlet. The interactive nature of the shell, easy to discover and use commands made learning Windows PowerShell very intuitive. I fell in love with the blue console. Whenever I learnt a new product or a technology, I would start playing around with its PowerShell module. As more and more goodies were incorporated into PowerShell, I adorned the researcher’s hat and started dwelling more and more into it. I began my day with my regular dose of caffeine and reading up on the various RSS feeds of the PowerShell blogs that I had bookmarked. It goes without saying that [PowerShell Magazine][2] has been my favorite. I became so passionate about Windows PowerShell that I was entitled “the PowerShell Evangelist” within my company even though my official designation was that of a consultant. I was busy delivering PowerShell webinars, mastering PowerShell classes (the favorite being the &#8220;Be the Bruce Almighty of Windows PowerShell&#8221;) and TechReady sessions that had PowerShell as its key ingredient. I started blogging on my TechNet blog at <http://powershell.ms>

One of the best things that happened to me was to be a member of the Bangalore PowerShell User Group ([PSBUG][3]), a brainchild of [Ravikanth Chaganti][4]&#8211;one of my role models. He has been a constant source of inspiration for me and his passion for Windows PowerShell is contagious. I remember walking up to him at MS TechEd sessions and asking for tips and tricks to master PowerShell. I also got to meet my other peers who shared the same zeal for PowerShell like [Dexter][5], [Pradeep][6], [Vinith][7], [Sahal][8], and [Harshul][9] to name a few.

After I’ve resigned Microsoft Services India and immigrated to Australia to join Dimension Data, Sydney, I realized most of the IT workforce here wasn’t aware of Windows PowerShell and if they were, they hated it. They attributed Windows PowerShell as a constraint being forced down the throats of IT Pros by the folks at Redmond. I realized that there is an opportunity as well as a challenge for me here. To help IT Pros in the Aussie territory to embrace Windows PowerShell. Fortunately I work as a Microsoft trainer. It is one of the most difficult jobs to execute. You get evaluated every week, unlike other jobs where an appraisal happens on a yearly or half-yearly basis. You need to stay on top of the technology you teach and back your knowledge with real world experience else you would be ripped apart in the evaluations on Friday evening. Every Microsoft course I taught, I converted a few modules to a PowerShell teach.

Obviously, that wasn’t part of the course, but that little “tease” of PowerShell was enough for them to get that “fear” or “hatred” out of PowerShell. My inbox started flooding with emails asking for help on how to get started with Windows PowerShell, do you teach “PowerShell” courses, if yes, when, do we have a local community for PowerShell etc. With such kind of response and the awesome support from Dimension Data Learning Solutions, I started organizing PowerShell meetings, online and at the office premises. I started writing extra modules that complement the current Microsoft courses but fulfill the automation piece that most of delegates wanted to achieve at their workplace.

Gradually, I saw a positive drift towards the PowerShell momentum in Sydney and hope it’ll spread across Australia. My vision is to make Windows PowerShell as popular as National Rugby League, Australian Football League or MasterChef Australia :). I know it is a daunting task but with focus towards Service Management Automation and Cloud, PowerShell isn’t just a “good to have” skill but now a “must to have” skill. We had meeting focused on System Center where I had the opportunity to evangelize the role of Windows PowerShell. Now, we have started Windows PowerShell meetings where the experience I gained during my PSBUG meetings has helped me strengthen the community here in Sydney. I have already started work on a few books and video tutorials on Windows PowerShell. Since I am no longer a Microsoft employee, I am transitioning my PowerShell blog from the TechNet blogs platform to [WordPress][10] and it’s a work in progress.

I was recently awarded the Microsoft MVP award for my work with Windows PowerShell. Honestly, this has really given me a boost in my efforts to help the community realize the true potential of Windows PowerShell. I would like to thank Microsoft for bestowing me with the coveted title and I am looking forward to work with fellow MVPs to help spread the word about the Windows PowerShell.

PS I would be presenting a seminar “Why WILL Windows PowerShell be a part of your future” on August 28 at DDLS, Sydney. More on this later on my blog and on DDLS website.

[1]: http://blogs.technet.com/b/manojnair/archive/2010/12/27/adding-users-to-ad-group-using-quest-powershell-command-lets.aspx
[2]: http://powershellmagazine.com
[3]: http://facebook.com/groups/psbug
[4]: http://www.ravichaganti.com/blog
[5]: https://twitter.com/DexterPOSH
[6]: https://www.facebook.com/pradeeprawat85
[7]: https://twitter.com/vinithmenon28
[8]: https://twitter.com/GetExchange
[9]: https://www.facebook.com/harshulhpatel
[10]: http://powershell.ms/