---
title: Day 1 From the PowerShell Deep Dive at TEC 2012
author: Steven Murawski
type: post
date: 2012-05-01T15:44:05+00:00
url: /2012/05/01/day-1-from-the-powershell-deep-dive-at-tec-2012/
views:
  - 8049
categories:
  - News
tags:
  - News
  - Conferences

---
Day 1 has wrapped up on the PowerShell Deep Dive and we are getting ready to start Day 2.  Day 1 was jam packed with PowerShell goodness.

We started out the day with [Jeffrey Snover][1]&#8216;s keynote highlighting the evolution of PowerShell in its march to V3 and how the customer focus has driven the team to delivering a tool which can magnify our ability to implement solutions and a peek at what is to come.

Next up was [Tome Tanasovski][2], diving deep into P/Invoke and opening up new realms of possibilities for interacting with the Win32 APIs.  Tome did a great job highlighting the various tips and tricks required to make P/Invoke work, and I really latched on to his explanation on how passing arguments by reference was important for P/Invoke and the importance of the Stringbuilder class.

[Kirk Munro][3] kept things rolling by opening our minds to the power of Proxy Functions and highlighted the [PowerShell Proxy Extensions][4] project to help us bend PowerShell commands to our will and workflow.  Kirk opened our eyes to the metaprogramming capabilities built into the core of PowerShell and how we can create rich wrappers for commands that maintain the consistency of the commands, but add (or remove) functionality.

[Brandon Shell][5] continued the roll by exposing us to the capabilities of the [PowerShell ResKit for Splunk][6] and the unique integrations that enables.  He really blew our minds with the real-time analytics that can be plumbed with PowerShell.

Krishna Vutukuri stepped up for the PowerShell team to bring us up to speed on the changes in PowerShell remoting in V3.  The portability of PowerShell sessions (Disconnect/Connect) really changes the PowerShell remote management game.

Finally, [Adam Driscoll][7] brought the Virtualization track and the PowerShell tracks together to highlight the new Hyper-V commands and capabilities in Windows Server 2012.  One of the key new features that Adam highlighted was Hyper-V Replica and the relatively few commands that are needed to make that functionality just work.

I can&#8217;t wait for today&#8217;s sessions!

[1]: http://www.jsnover.com
[2]: http://powertoe.wordpress.com/
[3]: http://poshoholic.com/
[4]: http://pspx.codeplex.com/
[5]: http://bsonposh.com/
[6]: http://dev.splunk.com/view/splunk-powershell-resource-kit/SP-CAAADRU
[7]: http://csharpening.net