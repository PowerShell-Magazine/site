---
title: 'PowerShell: The Great Problem Solver'
author: Andrea Fisher
type: post
date: 2012-03-28T18:00:58+00:00
url: /2012/03/28/powershell-the-great-problem-solver/
views:
  - 8242
post_views_count:
  - 1241
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
I began a love affair with scripting eight years ago. Fairly new to IT, I took every training opportunity I could. One of the security guys I knew started a user group to teach Perl. He claimed he simply wanted to evangelize the joys of scripting but I think he was just really tired of writing scripts for everyone else.

Back in 2002, Microsoft released a new scripting language called Monad. It was very similar to Perl so as Monad morphed into PowerShell, I quickly became a fan. I began using it for simple things; changing the local admin password on a group of servers, checking to see if a critical service was started on a set of servers. Every day I found a new way to use it. Over the years, I can’t tell you how many times I’ve used PowerShell to save the day.

Many people question the value of scripting but I learned quickly that there are numerous benefits.

You can use PowerShell to:

  * Easily query multiple systems for data
  * Complete tasks quickly and consistently
  * Execute repetitive tasks
  * Perform a single step from the command line instead of eight steps in the GUI

In a large enterprise standardization is key to having a well-managed environment. PowerShell is one way to accomplish that uniformity.

_**DNS**_: PowerShell helps solve a consistency problem with DNS servers. Several new DNS admins did not understand that creating a zone on the primary DNS server did not automatically create the zone on the secondary and tertiary servers. Tired of hearing about the lack of these zones causing problems, I wrote a PowerShell script. The script compared the zones on all three servers and if there were any inconsistencies, an email was sent, showing the missing zone and which server it was missing from.

_**The Registry**_: The introduction of Windows Server 2008 to our environment also introduced IPv6 which is enabled by default. The networking team reported a few issues with IPv6 packets crossing the routers. From Microsoft&#8217;s perspective, IPv6 is a mandatory part of the Windows operating system and should not be disabled, so that wasn’t an option. A joint call between Microsoft and Cisco determined that our course of action was to change a registry key on all the Windows 2008 servers. This key would disable IPv6 on all tunnel interfaces. Several different methods were discussed for accomplishing this. We determined that the quickest and easiest way was PowerShell. A simple six-line script solved a difficult problem within a matter of minutes. PowerShell to the rescue yet again

_**File Shares**_: PowerShell also saved the day about a year or so ago. There was a compliance gap at our company regarding share permissions on servers. The auditors were concerned that some application administrators, who had rights to log on to servers, could manipulate permissions on sensitive shares. There were hundreds of servers and thousands of shares involved. How could we possibly close that gap without lots of time and effort? PowerShell came to the rescue. Using just a few WMI classes, we were able to enumerate every share on the servers in question and add a group with restricted access to the shares. We then placed the application admins in the restricted group and viola, the compliance problem was solved.

PowerShell has proved to be invaluable over and over again.

Please take the time to add this to your skill set. Don’t be intimidated. It’s a lot easier than it looks.