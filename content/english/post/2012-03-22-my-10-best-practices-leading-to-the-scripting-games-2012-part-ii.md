---
title: My 10 best practices leading to the Scripting Games 2012 – Part II
author: Ifiok Moses
type: post
date: 2012-03-22T18:00:47+00:00
url: /2012/03/22/my-10-best-practices-leading-to-the-scripting-games-2012-part-ii/
views:
  - 17708
post_views_count:
  - 3699
categories:
  - Scripting Games
tags:
  - Scripting Games

---
In the first part of this write-up we looked at five of the top ten learning points as we prepare for the scripting games this year, since games are really popular in all their different categories from programming games to even Casino games as the judi dadu online which is great for people wanting to make money online. We will continue by discussing remoting, error handling and outputting objects to the pipeline.

### Support the enterprise

Why do you want to automate if all you do is manage just one machine?  Although you will certainly work more efficiently by automating repetitive tasks on a single server, the real power of PowerShell comes when you can send that single command to thousands of devices in the network and it performs just as well.

There are two basic ways of achieving this: The classic WMI, and the PowerShell V2 remoting. Several articles have been written at the Hey ScriptingGuy blog. Don Jones has an article in Technet magazine that explains this; while a full eBook dedicated to PowerShell remoting can be found at the powershell.com site, <http://powershell.com/cs/media/p/4908.aspx>, written by some of the top MVPs and is absolutely free.

Writing a script that flexes its muscles in the ability to pipe in a list of devices from active directory and initiate a software install, or a thousand names from a CSV file to create accounts in active directory, or build a hundred VMs from data in a text file.

### Manage the errors

A splash of red on the PowerShell console is one of the most frustrating things in using PowerShell, especially after typing one of those long Exchange commands, and the message is not even the easiest to understand.

PowerShell itself offers the ability to trap the error and:

  * Write a more meaningful error message to the screen
  * Write the error to a log file
  * Write the error to event log

Errors are classified into two: terminating errors and non-terminating errors.

  * Terminating errors will stop the script from running if it is encountered, and these could be syntax errors
  * Non terminating errors would display the errors and continue executing the script.

Depending on how these are managed, meaning what error settings are used, or where we have written it to, these errors can turn out to the very useful in several ways. For example: Errors written to a log file can help in:

  * Debugging the code
  * Finding out what is happening in the network.

There are several Scripting Guys’ articles that can help you understand this better.

Look for error handling resources in the following locations:

  1. <http://blogs.technet.com/b/heyscriptingguy/archive/tags/error+handling/>
  2. <http://blog.usepowershell.com/category/deep-dive/error-handling/>
  3. <http://tfl09.blogspot.com/2009/01/error-handling-with-powershell.html>
  4. <http://outputredirection.blogspot.com/2010/04/powershells-trycatchfinally-and.html>

### Use native PowerShell commands

Two articles in the Hey Scripting Guys blog brought my attention to this. In these articles Ed Wilson continues to stress the importance of considering native PowerShell commands first and only go for other options, like .NET classes, where there is no native PowerShell cmdlets to achieve a particular task, or where a native command does not offer the flexibility required in getting the desired result

These two articles are:

  * Why Use .NET Framework Classes from Within PowerShell? &#8211; <http://blogs.technet.com/b/heyscriptingguy/archive/2012/02/11/why-use-net-framework-classes-from-within-powershell.aspx>
  * Use .NET Framework Classes to Augment PowerShell when Required &#8211; <http://blogs.technet.com/b/heyscriptingguy/archive/2012/02/12/use-net-framework-classes-to-augment-powershell-when-required.aspx>

### Output objects

According to Don Jones: If you output pure text from a script or function, you&#8217;re doing it WRONG: <http://www.windowsitpro.com/blog/powershell-with-a-purpose-blog-36/windows-powershell/powershell-part-12-139695>

Marco Shaw presents a good comparison of the two cmdlets – Write-Host and Write-Output. It goes a long way to explain that, although there are several instances where you’d want to write host, write output is still better at presenting output that persists in the pipeline. This article can be found here: <http://blogs.technet.com/b/heyscriptingguy/archive/2011/05/17/writing-output-with-powershell.aspx>.

There are several ways to output objects. This TechNet article outlines some of the methods. <http://technet.microsoft.com/en-us/magazine/hh750381.aspx>. The choice really is yours and it all depends on your ultimate aim.

Another article by Steven Murawski continues on the same note: <http://blogs.technet.com/b/heyscriptingguy/archive/2011/05/19/create-custom-objects-in-your-powershell-script.aspx>

There is yet another post by Boe Prox on custom objects: <http://learn-powershell.net/2012/03/02/working-with-custom-types-of-custom-objects-in-powershell/>

This is no shortage of information on the net about creating objects and these can be used to strengthen one’s ability to generate outputs from scripts as the option of writing to host may not be a very viable one.

### Answer the question

ScriptingGuy emphasized during the PowerScripting podcast the importance of sticking the requirement of the event. You may actually lose points for presenting a long and complicated code that completely deviates from required solution.

Each event, as scripting guy, explains has specific things it’s testing for. It is important to:

  * Read the instruction carefully
  * Understand what it specifies
  * Utilize the simplest approach to solving the problem
  * As much as possible focus on native PowerShell commands, unless there are none.
  * If you require extra points – you may attempt to adjust your script to satisfy the extra conditions.

And that is final part of the write-up. Thanks for reading. I use this opportunity to wish everyone success in the scripting games 2012.
