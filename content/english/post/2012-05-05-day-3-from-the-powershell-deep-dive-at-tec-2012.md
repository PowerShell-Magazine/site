---
title: Day 3 From the PowerShell Deep Dive at TEC 2012
author: Steven Murawski
type: post
date: 2012-05-05T12:25:54+00:00
url: /2012/05/05/day-3-from-the-powershell-deep-dive-at-tec-2012/
views:
  - 5328
post_views_count:
  - 868
categories:
  - News
tags:
  - News
  - Conferences

---
On the third and final day of the PowerShell Deep Dive, the great content continued to roll.

The morning started with a bang when [Richard Siddaway][1] opened up with an overview of CDXML and how, using an XML configuration file, one can generate cmdlets from a WMI interface.  Kenneth Hansen, from the PowerShell team, made sure that our minds were sufficiently churning with ideas for using this new feature before sharing that CDXML was not limited to WMI, but that there is an API that allows CDXML to target almost any other source!  The face of unified management is changing, and we get to watch it unfold.

Our own <a title="Follow Aleksandar on Twitter" href="http://twitter.com/alexandair" target="_blank">Aleksandar Nikolic</a> covered how to build a constrained endpoint in PowerShell V3.  One off the challenges to the new constrained endpoints is how to allow native executables in the remote session.  Aleksandar shared some tips and tricks (like how to add the file system provider so that the shell can navigate to the location of the executable).

I was up next, and talked about PowerShell Workflows.  PowerShell Workflows are a very important new feature, but there are a number of tricky points.  One key thing to remember from PowerShell Workflows is, while they look like PowerShell, they are not only PowerShell, but also Windows Workflow Foundation workflows.  This means that scoping and variable access have some differences from a PowerShell script or module.

[Jeff Hicks][2] continued with some great tips for Office automation with PowerShell.  He showed us how to use the macro recorder to get an idea of the commands necessary when working with the Office COM objects and gave us a great tip on storing the values of some standard named constants used by Office in a custom object or hashtable for easy retrieval.

The last session of the day was Bruce Payette, the lead language designer.  Bruce talked in further depth about PowerShell Workflows and the underlying architecture for them.  He stressed that PowerShell Workflows be used for an appropriate purpose, like deployment and configuration, and not for things like data aggregation.

The PowerShell Deep Dive was a great event full of excellent content.  If you have not been to the Deep Dive before, you might not realize that the sessions are just the start of the learning here.  The conference attendees are all passionate about building solutions and finding better ways to make themselves more efficient.  The conversations before and after the regular day&#8217;s activities and between sessions is often as informative as the sessions themselves.  In large part, this is due to the great spirit of community around PowerShell that drives its members to share what they&#8217;ve learned.  If you have the opportunity to get to this event or one like it, it is well worth the investment of  your time and effort to be a part of the conversation.

&nbsp;

[1]: http://richardspowershellblog.wordpress.com/
[2]: http://jdhitsolutions.com/blog/