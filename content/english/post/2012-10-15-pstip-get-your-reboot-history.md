---
title: '#PSTip Get your reboot history'
author: Jakub Jareš
type: post
date: 2012-10-15T18:00:20+00:00
url: /2012/10/15/pstip-get-your-reboot-history/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Have you ever wondered how often is your station rebooted? Let&#8217;s ask the Windows Event Log and get time of last five reboots. You will use the Get-WinEvent cmdlet to connect to System event log. You are interested in &#8220;The Event log service was started.&#8221; event which has Id 6005. Let&#8217;s build a nice little XML query using here-string:

```
$xml=@'
<QueryList>
	<Query Id="0" Path="System">
		<Select Path="System">*[System[(EventID=6005)]]</Select>
	</Query>
</QueryList>
'@

PS> Get-WinEvent -FilterXml $xml -MaxEvents 5
   ProviderName: EventLog
TimeCreated                     Id LevelDisplayName Message
-----------                     -- ---------------- -------
10/8/2012 2:12:32 PM          6005 Information      The Event log service was started.
10/8/2012 10:52:34 AM         6005 Information      The Event log service was started.
10/8/2012 9:56:53 AM          6005 Information      The Event log service was started.
10/5/2012 11:53:52 AM         6005 Information      The Event log service was started.
10/4/2012 4:30:08 PM          6005 Information      The Event log service was started.
```

There is of course a 6006 event if you are more interested in shutdowns.

Another thing to notice is that the cmdlet itself implements a possibility to access event log on remote computers, making it an ideal tool for creating remote statistics.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-WinEvent -FilterXml $xml -MaxEvents 5 -ComputerName Server01,Server02
</pre>