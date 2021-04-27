---
title: Converting to size units (KB, MB,GB,TB, and PB) without using PowerShell multipliers
author: Ravikanth C
type: post
date: 2013-05-20T18:42:56+00:00
url: /2013/05/20/converting-to-size-units-kb-mbgbtb-and-pb-without-using-powershell-multipliers/
categories:
  - Brainteaser
tags:
  - BrainTeaser

---
Hello everyone! After some time, here is the brand new brain teaser. We hope you will like the challenge. 🙂 

In PowerShell, we can convert from bytes to KB, MB, GB, TB, and PB using the multipliers. For example,

<pre class="brush: powershell; title: ; notranslate" title="">$size = 123456789
$size / 1KB
$size / 1MB
$size / 1GB
$size / 1TB
$size / 1PB
</pre>

Now, here is a task for you. You need to find a way to perform the above conversion **without using any of the above PowerShell multipliers that is KB, MB, GB, TB, and PB**. Here are some more rules:

1. If the result contains decimal point, round the number down. For example: 12.99 should become 12.
  
2. All versions of PowerShell are allowed, shortest way wins.

We teamed up with Packt Publishing to offer the winner of this contest a copy of [Windows Server 2012 Automation with PowerShell Cookbook][1] eBook.

Here is a quick overview of this book:

• Extend the capabilities of your Windows environment

• Improve the process reliability by using well defined PowerShell scripts

• Full of examples, scripts, and real-world best practices

This contest closes by Saturday, May 25. Wear your PowerShell wizard hat and post your answers in the Comments section. The winner will be announced next Monday.

[1]: http://www.packtpub.com/windows-server-2012-automation-with-powershell/book