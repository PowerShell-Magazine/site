---
title: Helping Admins Getting Started with PowerShell
author: Jason Helmick
type: post
date: 2011-12-26T19:00:46+00:00
url: /2011/12/26/helping-admins-getting-started-with-powershell/
views:
  - 7569
post_views_count:
  - 1325
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
Writing for me has always been a daunting and seemingly endless task, one filled with pain. This is not due to the typical ‘writer’s block’ as I am full of ideas (sometimes referred to as bullshit from my friends), but the labor of the language itself has always challenged me. My inability to spell, incomplete and not well-formed sentences, along with things that dangle, I believe they are called participles.

Recently, someone I respect highly told me to grow-up and learn these relatively simple concepts so that I could communicate my ideas better. Ideas that he admits may have some value to others. His kindness is appreciated, even when wrapped in his ‘snarky’ ways.

The reason I even mention my deficiency, and determination, to overcome my affliction, is that I suspect that many administrators feel similar about PowerShell. Put yourself in the position of a Windows Admin, one that has spent their entire career in the safety of clicking a set of rehearsed instructions. This admin, who’s entire livelihood hinges on learning the next new button sequence is suddenly thrust into using a new command line interface called PowerShell. Listening to Admins after they see PowerShell for the first time produces the typical list of phrases:

  * Command line?  Isn’t that a step backwards?
  * New special commands called cmdlets? Great.
  * I hate learning syntax, are you sure there is no graphical way?
  * Looks like programming, I don’t want to be a programmer.
  * Pipelines? I got into Windows because I didn’t want to be a UNIX admin!

These comments would be laughable to the PowerShell elite if it were not for the underlying reason for them. Talk to an Admin that is trying to get started using PowerShell and what you find is fear. Fear that this will be painful, that they will not be successful, fear that they will lose their job without it. Do you remember learning your first command line interface? Learning a command line interface for the first time is a daunting and seemingly endless task. A prompt, a blinking prompt, blinking in condemnation, blinking in expectation of your next typed command, a prompt that is very unforgiving of syntax mistakes; this is a hard to learn tool.

So, how do we get Admins over the fear and start working with PowerShell? As a community, take time from our litany of bit-twiddling ‘coding’ blogs, and illuminate how a single one-liner could produce similar results. Perhaps take the giant wealth of knowledge that this community has and focus it for just a few moments of building great solutions with one-liners and automation that an Admin can use on the job. Maybe even go so far as to remind new Admins that PowerShell is an interactive command line tool that has some scripting capabilities, and that you don’t need to become a programmer to be hugely successful in your career. Maybe we should even invite our UNIX brethren to join us, if for no other reason than to keep us honest about what the job of an administrator truly is and how PowerShell fits into the picture (OK, maybe I just went too far). 😉

If you’re an Admin reading this, understand that the community of early-adopters is brilliant, caring, and always willing to help. Understand that you do not need to learn to be a developer and much of what you see on the web is the passion that we have over an extremely flexible and powerful tool. If you’re just getting started, ignore all the ‘code’ and focus on these things:

  1. Use PowerShell anytime you would usually use a command prompt.  Use it for ping, ipconfig, dir, launching Notepad, whatever the case might be. By doing this, you will find that PowerShell is not a programming language, but an interactive tool.
  2. Learn to read and use the help system. Help, Get-Help, Man, they all do the same thing, they get you help. PowerShell has the most extensive and useful help system of any command line. Good people worked very hard on this. It will solve your problem, but you have got to learn how to understand what it provides. Always, always use the –Full option. If you see something you don’t understand in the help system, don’t wait. Find out what it means.
  3. Learn how the pipeline works. I mean really learn how it works and how cmdlets pass information down the pipeline. You must be a master at understanding how cmdlets hook-up. This will allow you to discover how a cmdlet works (even ones you have never used before) and to make one-liners to solve problems.

If you begin your PowerShell journey by only focus on these three things, then you will quickly be using PowerShell for real solutions in your job. You can add the programmatic ‘code’ stuff such as automation and toolmaking later as it’s needed.

Where do you learn this? I will recommend to you the same thing I recommend to my closest friends just starting out with PowerShell. I will offer two options based on how you like to learn things.

  1. A book called ‘Learn Windows PowerShell in a Month of Lunches’ by Don Jones. Don’t get any other books yet until you accomplish the above three goals. This book will achieve those. Here’s a hint on how to use his book. Do exactly what he says to ‘do’. Don’t just read it, have a PowerShell prompt open and do exactly what he says.
  2. Take a class that focuses on the above tasks, not on scripting. I’m pretty sure I know of one or two classes that will work. 😉

Before I return to my grammar lessons, a last note to Admins beginning with PowerShell —you can be successful with PowerShell. You have a community of the brightest and best people I have ever met. Join the community and ask questions.

Knowledge is PowerShell,

Jason