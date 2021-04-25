---
title: Convert a string to a character array
author: Shay Levy
type: post
date: 2012-11-05T17:00:32+00:00
url: /2012/11/05/convert-a-string-to-a-character-array-2/
categories:
  - Brainteaser
tags:
  - Brainteaser

---
Hello everyone! We are back with a series of Brain Teasers. In the next 3 weeks we will publish one teaser per week.

It will run until Friday and the winner will be announced on the next Monday, taking home the eBook version of <a href="http://www.packtpub.com/microsoft-windows-powershell-3-0-firstlook/book" target="_blank">Microsoft Windows PowerShell 3.0 First Look</a> written by [Adam Driscoll][1]. We would like to thank our sponsor <a title="Packt" href="http://www.packtpub.com/" target="_blank">Packt</a>, one of the most prolific and fast-growing tech book publishers in the world, for providing such a cool prize.

OK, time to start your engine, here goes the first one 🙂

You have a string, &#8220;PowerShell&#8221;, you need to break it to its individual characters so the result is a collection of System.Char objects:

P

o

w

e

r

S

h

e

l

l

#### Requirements

1. You cannot cast the string to char array, e.g [char[]]&#8221;PowerShell&#8221;
  
2. You cannot use the String.ToCharArray method

The most shortest/elegant solution wins.

Good luck!

[1]: http://csharpening.net/