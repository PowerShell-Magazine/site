---
title: Day 2 From the PowerShell Deep Dive at TEC 2012
author: Steven Murawski
type: post
date: 2012-05-03T04:07:46+00:00
url: /2012/05/03/day-2-from-the-powershell-deep-dive-at-tec-2012/
views:
  - 5851
post_views_count:
  - 1046
categories:
  - News
tags:
  - News
  - Conferences

---
On the second day of the PowerShell Deep Dive, we had another great lineup of presentations.

The day started with [Jim Truher][1] describing a technique for creating formatting files for using Format-Custom.  He created a templating engine and used that to generate the xml formatting files.  Very cool stuff.  Watch his blog for the code.

Next on the agenda was [Kirk Munro][2].  Kirk talked about the experience of using WMI and the rich capabilities provided, but noted that the discovery experience for properties and methods was lacking.  He introduced a project that will be published (watch his blog) called WMIX (or the WMI Extensions).  This  project generates functions to wrap the calls to any WMI classes you desire.  It also uses the metadata from WMI to create objects that are more representative of the objects.

[James Brundage][3] illustrated how PowerShell can be used as a web language with [Pipeworks][4].  Not only can you build web pages and web applicatiions with Pipeworks.  Pipeworks also enables you to convert commands and modules to a software service.

I was up after James and went through my experiences, thoughts, and considerations as I implemented PowerShell V3 as part of my Windows Server 2012 implementation.

Travis Jones, from the PowerShell team, reviewed the enhancements in the jobs infrastructure with PowerShell V3.  Jobs now have a classification (since there are so many types), and include some nice new data, including start time and end time.

[Tome Tanasovski][5] covered considerations for developing a corporate module repository.  One interesting aspect of that talk was the discussion of versioning.  Other considerations included access to the repository, who has the ability to publish to the repository, and whether or not you have development guidance for modules.

The last presentation of the day was a talk about FIM and PowerShell.  Unfortunately, I missed that presentation.

We ended the day with a script party, where we had three tables of people discussing problems and solutions, working through code, and learning from each other.  What a great end to a great day.

[1]: http://jtruher3.wordpress.com/
[2]: http://poshoholic.com
[3]: http://www.start-automating.com
[4]: http://powershellpipeworks.com
[5]: http://powertoe.wordpress.com