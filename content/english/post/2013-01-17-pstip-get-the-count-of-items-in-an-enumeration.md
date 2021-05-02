---
title: '#PSTip Get the count of items in an enumeration'
author: Ravikanth C
type: post
date: 2013-01-17T19:01:49+00:00
url: /2013/01/17/pstip-get-the-count-of-items-in-an-enumeration/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

I recently got a couple of [Blink(1)][1]s and started exploring how I can use them in PowerShell. If I have to describe blink(1) in a single line, it is a USB LED that responds to programmed events by blinking in whatever color we specify.

While working with this, I was exploring how I can use the [[System.Drawing.KnownColor]][2] enumeration. So, one of the items on my list was to find the count of items in this enumeration.

So, this is how I did that:

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName System.Drawing
[Enum]::GetValues([System.Drawing.KnownColor]).Count
</pre>

In the next tip, I will show how I used this count value to retrieve a random item from the enumeration.

[1]: http://thingm.com/products/blink-1.html
[2]: http://msdn.microsoft.com/en-us/library/system.drawing.knowncolor.aspx