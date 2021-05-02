---
title: '#PSTip Parsing Exchange size strings'
author: Shay Levy
type: post
date: 2013-10-08T18:00:56+00:00
url: /2013/10/08/pstip-parsing-exchange-size-strings/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
You have been tasked to generate a report of mailbox sizes and you quickly write the following:

```
Get-MailboxDatabase |
Get-MailboxStatistics |
Select-Object DisplayName,TotalItemSize |
Export-Csv .\MailboxSize.csv -NoTypeInformation
```


A week after, your manager asks you to re-format the size to another size unit (KB,MB,GB, and so on). You open the file and see that the size of each mailbox is looking similar to: _1006 MB (1,055,195,632 bytes)_. See the problem?

The size you have in the CSV file is a string and you need to gather your string parsing-fu to convert it; neither an easy nor fun task! Luckily, Exchange has a special type, _ByteQuantifiedSize_, you can use to parse the string and it even allows you to convert the result between the size units. Note that the _ByteQuantifiedSize_ Type is available only in the Exchange Management Shell (EMS), it will not work in an implicit remoting session made to the PowerShell IIS directory on your Exchange server (thanks [@mjolinor][1]).

```
PS> [Microsoft.Exchange.Data.ByteQuantifiedSize]::Parse('1006 MB (1,055,195,632 bytes)')
1006 MB (1,055,195,632 bytes)
```


At first glance, nothing happened when you parsed the string but what you see on screen is just the string representation of the size object. Here are the interesting bits&#8230;

```
PS> [Microsoft.Exchange.Data.ByteQuantifiedSize]::Parse('1006 MB (1,055,195,632 bytes)') | Get-Member
   TypeName: Microsoft.Exchange.Data.ByteQuantifiedSize
Name          MemberType Definition
----          ---------- ----------
CompareTo     Method     int CompareTo(Microsoft.Exchange.Data.ByteQuantifiedSize other)
Equals        Method     bool Equals(System.Object obj), bool Equals(Microsoft.Exchange.Data.ByteQuantifiedSize other)
GetHashCode   Method     int GetHashCode()
GetType       Method     type GetType()
RoundUpToUnit Method     System.UInt64 RoundUpToUnit(Microsoft.Exchange.Data.ByteQuantifiedSize+Quantifier quantifier)
ToBytes       Method     System.UInt64 ToBytes()
ToGB          Method     System.UInt64 ToGB()
ToKB          Method     System.UInt64 ToKB()
ToMB          Method     System.UInt64 ToMB()
ToString      Method     string ToString(), string ToString(string format), string ToString(string format, System.IF...
ToTB          Method     System.UInt64 ToTB()
```

Notice the _To*_ methods. We can use them to convert to other units.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $size = [Microsoft.Exchange.Data.ByteQuantifiedSize]::Parse('1006 MB (1,055,195,632 bytes)')
PS&gt; $size.ToKB()
1030464
</pre>

[1]: https://twitter.com/mjolinor