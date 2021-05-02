---
title: '#PSTip How to configure storage quotas for a mailbox using PowerShell'
author: Shay Levy
type: post
date: 2013-09-09T18:00:51+00:00
url: /2013/09/09/pstip-how-to-configure-storage-quotas-for-a-mailbox-using-powershell/
categories:
  - Exchange
  - Tips and Tricks
tags:
  - Exchange
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Exchange lets us configure mailbox storage quotas for a mailbox, control the size of mailboxes and manage the growth of mailbox databases. Storage quotas can be configured on a per-database basis or explicitly on a mailbox level. When a mailbox size reaches or exceeds a specified storage quota limit, Exchange sends a descriptive notification to the mailbox owner.

We can set storage quota limits for the following fields:

  * **Issue warning at (KB)** : Specifies the maximum storage limit in kilobytes (KB) before a warning is issued to the mailbox user. The value range is from 0 through 2,147,483,647 KB. If the mailbox size reaches or exceeds the value specified, Exchange sends _a warning message_ to the mailbox user.
  * **Prohibit send at (KB)** : Specifies a prohibit send limit in KB for the mailbox. The value range is from 0 through 2,147,483,647 KB. If the mailbox size reaches or exceeds the specified limit, Exchange prevents the mailbox user from sending new messages and displays _a descriptive error message_.
  * **Prohibit send and receive at (KB)** : Specifies a prohibit send and receive limit in KB for the mailbox. The value range is from 0 through 2,147,483,647 KB. If the mailbox size reaches or exceeds the specified limit, Exchange prevents the mailbox user from sending new messages and won&#8217;t deliver any new messages to the mailbox. Any messages sent to the mailbox are returned to the sender with a descriptive error message.

To set these limits explicitly on a mailbox object, use the _Set-Mailbox_ cmdlet. Launch the Exchange Management Console (EMC) and type the following command:

<pre class="brush: powershell; title: ; notranslate" title="">Set-Mailbox -Identity jsmith -IssueWarningQuota 1.7 -ProhibitSendQuota 1.9gb -ProhibitSendReceiveQuota 2gb -UseDatabaseQuotaDefaults $false
</pre>