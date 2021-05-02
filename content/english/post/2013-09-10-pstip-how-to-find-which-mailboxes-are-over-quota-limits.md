---
title: '#PSTip How to find which mailboxes are over quota limits'
author: Shay Levy
type: post
date: 2013-09-10T18:00:28+00:00
url: /2013/09/10/pstip-how-to-find-which-mailboxes-are-over-quota-limits/
categories:
  - Exchange
  - Tips and Tricks
tags:
  - Exchange
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In a previous tip we saw [how to configure storage quotas for a mailbox using PowerShell][1]. Today we will see how we can find all mailboxes that are over quota or received a quota message. Let&#8217;s check the quota status for a given mailbox. Open the Exchange Management Console (EMC) and type the following commands:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $stats = Get-Mailbox user1 | Get-MailboxStatistics
PS&gt; $stats.StorageLimitStatus
BelowLimit
</pre>

You can see that the quota limit has not been reached and that the mailbox capacity is below the limit. _StorageLimitStatus_ is a _System.Enum_ object that holds several values:

```
PS> $stats.StorageLimitStatus.GetType().FullName
Microsoft.Exchange.Data.Mapi.StorageLimitStatus

PS> [enum]::GetNames('Microsoft.Exchange.Data.Mapi.StorageLimitStatus')
BelowLimit
IssueWarning
ProhibitSend
NoChecking
MailboxDisabled
```

We can check the status of all mailboxes on a given database:

```
PS> Get-Mailbox -Database db1 | Get-MailboxStatistics | Select-Object DisplayName,StorageLimitStatus
DisplayName         StorageLimitStatus
-----------         ------------------
User1                       BelowLimit
User2                     IssueWarning
User3                       BelowLimit
(...)
```

Or for all mailboxes. You may want to refine the search to return just the mailboxes with a specified _StorageLimitStatus_ value (over quota).

```
PS> Get-MailboxDatabase | Get-MailboxStatistics | Where-Object {$_.StorageLimitStatus -match 'IssueWarning|ProhibitSend|MailboxDisabled'}

DisplayName       ItemCount StorageLimitStatus        LastLogonTime
-----------       --------- ------------------        -------------
User2             2400            IssueWarning
User4             5010            IssueWarning 9/8/2013 10:04:20 AM
User5             2956            IssueWarning 9/8/2013 10:03:46 AM
User6             2316         MailboxDisabled  9/8/2013 9:53:11 AM
User7             4433            ProhibitSend
```

**Note**: To check the status of archive mailboxes, add the &#8211;_Archive_ switch to the _Get-Mailbox_ command.

[1]: /2013/09/08/pstip-how-to-configure-storage-quotas-for-a-mailbox-using-powershell