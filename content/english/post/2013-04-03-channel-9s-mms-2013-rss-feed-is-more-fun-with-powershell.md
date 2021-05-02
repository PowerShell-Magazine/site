---
title: Channel 9’s MMS 2013 RSS feed is more fun with PowerShell
author: Aleksandar Nikolic
type: post
date: 2013-04-03T19:59:33+00:00
url: /2013/04/03/channel-9s-mms-2013-rss-feed-is-more-fun-with-powershell/
categories:
  - News
tags:
  - News

---
The <a href="http://www.2013mms.com/" title="MMS 2013" target="_blank">MMS 2013</a> conference starts next week. The keynote and sessions will be available for replay through the Channel 9 web site. Channel 9 has unveiled <a href="http://channel9.msdn.com/Events/MMS/2013" title="The Channel 9 MMS 2013 page" target="_blank">their MMS 2013 page</a> including an <a href="http://channel9.msdn.com/Events/MMS/2013/RSS" title="Channel 9's MMS 2013 RSS Feed" target="_blank">RSS feed</a>. We can use the RSS feed and PowerShell to easily browse the session catalog. 

The following command uses the _Invoke-RestMethod_ cmdlet, introduced in Windows PowerShell 3.0, to get information from the MMS 2013 RSS feed. We select a few properties and pass them on to the _Out-GridView_ cmdlet. In a grid view, we can perform further filtering (picking up only sessions where a summary contains the word &#8220;powershell&#8221;, for example), select the sessions we are interested in, and, thanks to a new _-PassThru_ parameter, pipe them to the _Export-Csv_ cmdlet. 

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $rss = 'http://channel9.msdn.com/Events/MMS/2013/RSS'
PS&gt; Invoke-RestMethod -Uri $rss | select Title, Creator, Summary, Link | Out-GridView -PassThru | Export-Csv -Path c:\temp\mms2013.csv -Encoding UTF8
</pre>

This is how the grid view looks like after filtering PowerShell-related sessions (click for larger image):

![](/images/MMS2013_GridView.png)

Now that we have our sessions exported to a CSV file, we can import the file, and look at the details of some specific session that we are interested in:

```
PS> Import-Csv c:\temp\mms2013.csv | where title -match 'azure' | fl *

title   : Take Control of the Cloud with the Windows Azure PowerShell Cmdlets
creator : Michael Washam
summary : In this session you will learn how to tame the simplest to most advanced Windows Azure deployments with the Windows Azure PowerShell cmdlets. Automate repetitive tasks and learn 
          how build advanced reproducible deployments so you can save time and money while working in the cloud. The presenter will cover dev-ops scenarios with Virtual Machines and Cloud 
          Services in this demo packed session.
link    : http://channel9.msdn.com/Events/MMS/2013/WS-B311
```

