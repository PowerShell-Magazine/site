---
title: '#PSTip Working with DateTime objects in PowerShell using [System.DateTime]'
author: Jaap Brasser
type: post
date: 2012-10-04T18:00:36+00:00
url: /2012/10/04/pstip-working-with-datetime-objects-in-powershell-using-system-datetime/
post_views_count:
  - 4172
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When working with dates in PowerShell it is often useful to leverage the .NET Framework System.DateTime class. For example to display the current date and time the Now method can be used:

<pre class="brush: powershell; title: ; notranslate" title="">[System.DateTime]::Now
</pre>

[System.DateTime] can be shortened to [DateTime] as shown in the following example, where it is used incombination with the Today method to display the current date:

<pre class="brush: powershell; title: ; notranslate" title="">[DateTime]::Today
</pre>

When working with time stamps found in Active Directory the FromFileTime method can be used to convert the file time to a human readable format.

<pre class="brush: powershell; title: ; notranslate" title="">[DateTime]::FromFileTime(129955032000000000)
</pre>

Another very useful feature is the ParseExact method that is included in this class. It transforms a string into a DateTime object. It can be used to convert almost any type of date by specifying the format using the MM-dd-yyyy notation. Running the following command will create a DateTime object that display the date correctly as the 16<sup>th</sup> of May, 2005.

<pre class="brush: powershell; title: ; notranslate" title="">[DateTime]::ParseExact('16@05==5','dd@MM==y',$null)
</pre>

For more information about the DateTime methods available in this class, please refer to this MSDN article: <http://msdn.microsoft.com/en-us/library/497a406b.aspx>