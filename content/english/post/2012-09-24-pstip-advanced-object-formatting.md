---
title: '#PSTip Advanced object formatting'
author: Shay Levy
type: post
date: 2012-09-24T18:00:29+00:00
url: /2012/09/24/pstip-advanced-object-formatting/
views:
  - 9820
post_views_count:
  - 1708
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
With the Format-Table cmdlet we format and display a tabular format of an output, We can also select and display output of selected properties of objects.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process | Format-Table Name,StartTime
</pre>

You can also use a hash table to add calculated properties to an object before displaying it. The following command displays a table with the Name and StartTime of all processes on the local computer (not all processes have this property set). The StartTime is formatted using .NET format specifiers to display the date in mm/dd/yy format.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process | Format-Table Name,@{Name='StartDate';Expression={"{0:d}" -f $_.StartTime}}
</pre>

But there&#8217;s a better, built-in, PowerShell way to do this, one that doesn&#8217;t invole the traditional .NET way of string formatting (e.g &#8216;{0}&#8217;):

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process | Format-Table Name,@{Name='StartDate';Expression={$_.StartTime};FormatString="d"}
</pre>

Using the FormatString key (available in v2 and above), all you need to do is specify the modifier itself, PowerShell will do the rest.

And as you probably know, the hash table keys can be specified by just their first letter, so the following is also valid:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process | Format-Table Name,@{n='StartDate';e={$_.StartTime};f="d"}
</pre>

**Note**: Calculated properties can be used using the Select-Object cmdlet but unfortunately the FormatString cannot be used with it. I logged a suggestion so [add your vote][1] if you want to have this option in Select-Object.

[1]: http://connect.microsoft.com/PowerShell/feedback/details/568658/make-select-object-support-the-formatstring-key