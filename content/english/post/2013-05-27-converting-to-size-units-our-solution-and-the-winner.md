---
title: Converting to size units – our solution and the winner
author: Shay Levy
type: post
date: 2013-05-27T18:00:00+00:00
url: /2013/05/27/converting-to-size-units-our-solution-and-the-winner/
categories:
  - Brainteaser
tags:
  - Brainteaser

---
The task was to convert a number to the size units without using the built-in multipliers. Thank you all for taking the time to participate in this teaser.

Now, let&#8217;s present the solution we had in mind. Starting with PowerShell 3.0, we now have two new bitwise arithmetic operators: shift-left (_shl_) and shift-right (_shr_). You can read more about them in the [about\_Comparison\_Operators][1] help topic.

The solution to this teaser is using the _shift-right_ operator to convert the value of given number (representing a value in bytes).

Generally speaking, shifting right a number divides it by 2 and rounds the number down. Each division takes the result of the last operation and divides it again by 2.

```
PS> 100 -shr 1
50

PS> 100 -shr 2
25

PS> 100 -shr 3
12
```

Now, let&#8217;s take the value of 1KB: 1024. To get to 1024 we need to raise 2 in the power of 10. For 1MB (1048576), we need to raise it in the power of 20. 30 for _1GB_, 40 for _1PB_ and so on and so forth.

```
PS> [math]::Pow(2,10)
1024

PS> [math]::Pow(2,20)
1048576

PS> [math]::Pow(2,30)
1073741824
```

We can use the values 10, 20, 30&#8230; and shift-right by them to get the corresponding unit sizes in _KB, MB, GB&#8230;_. We will use the value of 13947906293.76 (that&#8217;s 12.99GB, actually).

```
$value = 13947906293.76

# get the value in KB
PS> $value -shr 10
13621002

# get the value in MB
PS> $value -shr 20
13301

# get the value in GB
PS> $value -shr 30
12
```

If you happen to manage Exchange 2007/2010 you are probably familiar with the size formatting _ToKB/ToMB/ToGB/ToTB_ methods. Actually I owe the idea for this teaser to the Exchange team&#8211;they calculate the requested size by shifting right the value.

The formatting methods are available on _[ByteQuantifiedSize][2]_ objects (_[Microsoft.Exchange.Data.ByteQuantifiedSize]_).

Here&#8217;s a list of _ByteQuantifiedSize_ properties of a Mailbox object:

```
(Get-Mailbox shay).PSObject.Properties |
Where-Object {$_.TypeNameOfValue -like "*ByteQuantifiedSize*"} |
Format-Table name

Name
----
ProhibitSendQuota
ProhibitSendReceiveQuota
RecoverableItemsQuota
RecoverableItemsWarningQuota
IssueWarningQuota
RulesQuota
ArchiveQuota
ArchiveWarningQuota
MaxSendSize
MaxReceiveSize
```

To format the size of _ByteQuantifiedSize_ property, you refer to its _Value_ property and then to one of the above methods:

```
PS> $mbx.ArchiveQuota
IsUnlimited Value
----------- -----
  False 50 GB (53,687,091,200 bytes)

PS> $mbx.ArchiveQuota.Value | Get-Member
   TypeName: Microsoft.Exchange.Data.ByteQuantifiedSize
Name          MemberType Definition
----          ---------- ----------
CompareTo     Method     int CompareTo(Microsoft.Exchange.Data.ByteQuantifiedSize other), int IComparable.CompareTo(...
Equals        Method     bool Equals(System.Object obj), bool Equals(Microsoft.Exchange.Data.ByteQuantifiedSize other)
GetHashCode   Method     int GetHashCode()
GetType       Method     type GetType()
RoundUpToUnit Method     uint64 RoundUpToUnit(Microsoft.Exchange.Data.ByteQuantifiedSize+Quantifier quantifier)
ToBytes       Method     uint64 ToBytes()
ToGB          Method     uint64 ToGB()
ToKB          Method     uint64 ToKB()
ToMB          Method     uint64 ToMB()
ToString      Method     string ToString(), string ToString(string format), string ToString(string format, System.IF...
ToTB          Method     uint64 ToTB()

PS> $mbx.ArchiveQuota.Value.ToGB()
50
```

Now to the prize. Congratulations Jaap Brasser and Rob Campbell, you get to take with you a copy of the [Windows Server 2012 Automation with PowerShell Cookbook][3] eBook. We would like to thank again [Packt Publishing][4] for the eBooks.

Jaap and Rob, in the best commmunity spirit, came up with the solution that was the shortest one that returns the right results for all five supported unit sizes:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $s = 12345678964561111
PS&gt; 1..5|%{[long](($s/=1024)-.5)}
12056327113829
11773756947
11497809
11228
10
</pre>

Let&#8217;s rewrite our solution in the same manner:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; 1..5|%{$s -shr 10*$_}
</pre>

Amazingly, we can even remove all spaces, and it&#8217;ll still work:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; 1..5|%{$s-shr10*$_}
</pre>

See you in the next brainteaser. 🙂

[1]: http://technet.microsoft.com/en-us/library/hh847759.aspx
[2]: http://msdn.microsoft.com/en-us/library/exchange/microsoft.exchange.data.bytequantifiedsize_members%28v=exchg.150%29.aspx
[3]: http://www.packtpub.com/windows-server-2012-automation-with-powershell/book
[4]: http://www.packtpub.com/