---
title: '#PSTip Transposing lines in PowerShell ISE'
author: Shay Levy
type: post
date: 2013-01-28T19:01:20+00:00
url: /2013/01/28/pstip-transposing-lines-in-powershell-ise/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

When working in the ISE editor you sometimes need to rearrange lines of code and move them below or above other lines. The natural process is to highlight a line, cut it, move your cursor position to another position and then paste.

In ISE 3.0 there&#8217;s a hidden keyboard shortcut that can save you a lot of key strokes and make the process a lot easier. Consider the following content. You want to move the first line two lines below.

![](/images/before.png)

All you need to do is place your cursor anywhere inside the first line and press the Alt+Shift+T key combination twice. Try it.

![](/images/after.png)

Unfortunately, you can only swap lines in one direction only, from top to bottom. There is no keyboard shortcut to do the opposite.