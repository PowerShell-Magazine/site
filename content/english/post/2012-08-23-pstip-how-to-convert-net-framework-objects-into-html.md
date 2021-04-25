---
title: '#PSTip How to convert .NET Framework objects into HTML'
author: Aleksandar Nikolic
type: post
date: 2012-08-23T18:00:33+00:00
url: /2012/08/23/pstip-how-to-convert-net-framework-objects-into-html/
views:
  - 6914
post_views_count:
  - 1206
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The ConvertTo-Html cmdlet converts Microsoft .NET Framework objects into HTML that can be displayed in a Web browser. In the following example we pick up a few properties of process objects with name that contains the word &#8216;host&#8217;, sort the output by the &#8216;Handles&#8217; property, convert it to HTML, and save it as HTML file in the user’s TEMP folder. At the end we open the HTML file in the default browser.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process -Name *host* |
Select-Object -Property Name,Id,Handles |
Sort-Object -Property Handles -Descending |
ConvertTo-Html -Title "PowerShell Rocks!" -Body "&lt;h1&gt;Info about *host* processes&lt;/h1&gt;" |
Out-File -FilePath $env:TEMP\processes.html
</pre>

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Item -Path $env:TEMP\processes.html
</pre>