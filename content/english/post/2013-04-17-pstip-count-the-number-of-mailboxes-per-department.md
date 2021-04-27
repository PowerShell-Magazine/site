---
title: '#PSTip Count the number of mailboxes per department'
author: Shay Levy
type: post
date: 2013-04-17T18:00:32+00:00
url: /2013/04/17/pstip-count-the-number-of-mailboxes-per-department/
categories:
  - Tips and Tricks
  - Exchange
tags:
  - Exchange
  - Tips and Tricks

---
Yesterday I got an email asking for help to create a report of all user mailboxes in Exchange per department. When you execute a _Get-Mailbox_ command, you&#8217;ll see that the _Department_ property is not included in the result.

```
PS> Get-Mailbox shay |  Get-Member d*
   TypeName: Microsoft.Exchange.Data.Directory.Management.Mailbox

Name                                 MemberType Definition
----                                 ---------- ----------
Database                             Property   Microsoft.Exchange.Data.Directory.ADObjectId Database {get;}
DeliverToMailboxAndForward           Property   bool DeliverToMailboxAndForward {get;set;}
DisabledArchiveDatabase              Property   Microsoft.Exchange.Data.Directory.ADObjectId DisabledArchiveDatabase...
DisabledArchiveGuid                  Property   guid DisabledArchiveGuid {get;}
DisplayName                          Property   string DisplayName {get;set;}
DistinguishedName                    Property   string DistinguishedName {get;}
DowngradeHighPriorityMessagesEnabled Property   bool DowngradeHighPriorityMessagesEnabled {get;set;}
```

The Department property is a part of the _Get-User_ cmdlet.

```
PS> Get-User shay | Format-Table Name,Department -AutoSize

Name      Department
----      ----------
Shay Levy Computers
```

The most common solution is to invoke the Get-User command for each mailbox object and grab its Department property:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Mailbox | Select-Object Name,@{n='Department';e={ ($_ |Get-User).Department}}
</pre>

But that requires executing two cmdlets to get the information. We can generate a quick report of users count per department:

<pre class="brush: powershell; title: ; notranslate" title="">Get-User -ResultSize Unlimited | Group-Object Department -NoElement
</pre>

What about mailboxes? You can try and filter the results of the above command to include just user mailboxes using a _Where-Object_ command, but a better way would be to use one of the parameters of the _Get-User_ cmdlet and filter the objects as early as you can.

<pre class="brush: powershell; title: ; notranslate" title="">Get-User -ResultSize Unlimited -RecipientTypeDetails UserMailbox |
Group-Object Department -NoElement | Sort-Object Count -Descending
</pre>