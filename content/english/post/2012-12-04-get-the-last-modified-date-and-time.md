---
title: Get the last modified date and time
author: Shay Levy
type: post
date: 2012-12-04T17:00:50+00:00
url: /2012/12/04/get-the-last-modified-date-and-time/
categories:
  - Brainteaser
tags:
  - Brainteaser

---
Before we dive into this week&#8217;s teaser we would like to announce the winner of last week&#8217;s challenge. **Ioan Corcodel**, congratulations! You take with you a copy of <a href="http://www.packtpub.com/microsoft-windows-powershell-3-0-firstlook/book" target="_blank">Microsoft Windows PowerShell 3.0 First Look</a> written by <a href="http://csharpening.net/" target="_blank">Adam Driscoll</a>. Once again, we would like to thank our sponsor <a href="http://www.packtpub.com/" target="_blank">Packt</a>, one of the most prolific and fast-growing tech book publishers in the world, for providing such a cool prize.

Ioan, your solution worked great and it was 78 characters long. By the way, I was able to shave off a few more characters 🙂 ,75 vs. 78 :

echo Hannah Jeffrey &#8216;12321&#8217; Madam Abracadabra|?{-join$\_[$\_.Length..0]-eq$_}

As for this week&#8217;s teaser, your task is to get the last modified date and time of a file system object, **without** referring to the _LastWriteTime_ property.

For this example, you need to emit the _LastWriteTime_ value of _powershell.exe_ as a System.DateTime object.

One thing though, **there&#8217;s no prize this week.** I know you guys are here for the challenge and the prize is only an excuse to participate and share your knowledge. 🙂

Again, comments are allowed until Friday and we will announce the winner on Monday, next week.

Please use the comment box at the bottom of this page to submit your solution. Don&#8217;t have a solution of your own, or it has been already posted by others? You can still participate and add your voice by voting on a existing comments, use the up/down voting arrows.