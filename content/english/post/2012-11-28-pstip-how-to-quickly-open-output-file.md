---
title: '#PSTip How to quickly open output file'
author: Aleksandar Nikolic
type: post
date: 2012-11-28T21:00:28+00:00
url: /2012/11/28/pstip-how-to-quickly-open-output-file/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

When you work interactively with PowerShell, you don&#8217;t want to type too much. You want to use aliases, positional parameters, redirection operators, automatic variables&#8230; You just want to type as little as possible. 🙂

Let&#8217;s start with the fully typed command. The command selects three properties of the process objects, converts output to a HTML page, and sends the resulting HTML page to the c:\temp\process.html file. You also specify a nice title for the HTML page.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Process | ConvertTo-Html -Property Name,Path,Company -Title "Process Information" | Out-File -FilePath c:\temp\process.html
</pre>

How can you make this command shorter? You can use gps (or even ps) alias instead of the Get-Process cmdlet, omit ConvertTo-Html&#8217;s -Property parameter (it&#8217;s a positional parameter), and use the redirection operator (>) instead of the Out-File cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; gps | ConvertTo-Html Name,Path,Company -Title "Process Information" &gt; c:\temp\process.html
</pre>

You have your HTML file now, but you also want to open it in the default browser and check the result. The following command might look cryptic, but I&#8217;m sure you will use it all the time after you finish reading this tip.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ii $$
</pre>

That command&#8217;s opened the c:\temp\process.html file in the default browser, right? The magic of PowerShell. Why does that command work? _ii_ is the alias for the Invoke-Item cmdlet, but the real power lies in the $$ automatic variable. $$ automatic variable contains the last token in the last line received by the session, and in this case that&#8217;s c:\temp\process.html.

By the way, _start_, the alias of the Start-Process cmdlet, comes in handy for this task as well.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; start c:\temp\process.html
</pre>

That&#8217;s short and easy to understand. However, as you have seen, you can do it better.