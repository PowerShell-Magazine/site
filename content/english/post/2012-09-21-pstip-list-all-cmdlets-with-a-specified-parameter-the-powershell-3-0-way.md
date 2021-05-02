---
title: '#PSTip List all cmdlets with a specified parameter – The PowerShell 3.0 way!'
author: Ravikanth C
type: post
date: 2012-09-21T18:00:49+00:00
url: /2012/09/21/pstip-list-all-cmdlets-with-a-specified-parameter-the-powershell-3-0-way/
views:
  - 6638
post_views_count:
  - 1275
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In Windows PowerShell 2.0, if we had to list a complete list of cmdlets that had a specific parameter, we could have done it in a couple of different ways:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Help * -Parameter ComputerName
</pre>

or we can examine the cmdlet meta returned by Get-Command cmdlet!

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command * | Where {$_.Definition -like "*-ComputerName*"}
</pre>

In PowerShell 3.0, Get-Command cmdlet provides a simple way to do this.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command -ParameterName ComputerName
</pre>