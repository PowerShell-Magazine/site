---
title: Find a list of all IP addresses assigned to the local system
author: Ravikanth C
type: post
date: 2012-11-19T18:20:38+00:00
url: /2012/11/19/find-a-list-of-all-ip-addresses-assigned-to-the-local-system/
categories:
  - Brainteaser
tags:
  - Brainteaser

---
Hello everyone! The Brain Teaser series continues.

First, we would like to announce the winner of <a href="http://104.131.21.239/2012/11/12/assign-a-list-of-processes-to-a-variable-and-output-it-to-the-console/" target="_blank">the previous brain teaser</a>. We got a few answers in which many of them have a command length of 8. The shortest answer &#8212; ($x=ps), with 7 characters &#8212; was given by Mike F Robbins. So, he is the winner! In his solution, the assignment is placed inside the parenthesis so that the results of &#8220;ps&#8221; (alias to Get-Process) are stored in the variable _$x_ as well as displayed on the console.

Congratulations Mike, you get an eBook version of <a href="http://www.manning.com/jones3/" target="_blank">Learn Windows PowerShell 3 in a Month of Lunches, Second Edition</a> written by <a href="http://donjones.com/" target="_blank">Don Jones</a> and <a href="http://jdhitsolutions.com/blog" target="_blank">Jeffery Hicks</a>.

Here is the new brain teaser!

Your new task is to get a list of IP addresses (IPv4 and IPv6, if present) of the local machine. Sounds easy? Well, here are some of the constraints to make it interesting:

  * You cannot use WMI
  * You cannot use IPCONFIG
  * You cannot reference $env:ComputerName or localhost as the computer name
  * Your command should return an IP address object!
  * You cannot use Windows 8 networking cmdlets or similar for other OS

<div>
  Once again, the shortest answer wins! Be aware, a space is a character too.
</div>

Please use the comment box at the bottom of this page to submit your solutions by Friday. The winner will be announced on the next Monday.

Don&#8217;t have a solution of your own or has it already been posted by others? You can still participate and add your voice by voting on the existing comment by using the up/down voting arrows.

This time, the prize is again the eBook version of <a href="http://www.packtpub.com/microsoft-windows-powershell-3-0-firstlook/book" target="_blank">Microsoft Windows PowerShell 3.0 First Look</a> written by [Adam Driscoll][1]. We would like to thank our sponsor <a title="Packt" href="http://www.packtpub.com/" target="_blank">Packt</a>, one of the most prolific and fast-growing tech book publishers in the world, for providing such a cool prize.

Good luck!

[1]: http://csharpening.net/