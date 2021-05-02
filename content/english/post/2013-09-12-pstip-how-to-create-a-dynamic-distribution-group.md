---
title: '#PSTip How to create a dynamic distribution group'
author: Shay Levy
type: post
date: 2013-09-12T18:00:27+00:00
url: /2013/09/12/pstip-how-to-create-a-dynamic-distribution-group/
categories:
  - Exchange
  - Tips and Tricks
tags:
  - Exchange
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

One of the group types that Exchange can create is a dynamic distribution group. Unlike regular distribution groups that contain a defined set of members, the membership list for dynamic distribution groups is calculated each time a message is sent to the group, based on the filters and conditions that you define.  When an email message is sent to a dynamic distribution group, it is delivered to all recipients in the organization that match the criteria defined for that group.

To create a dynamic distribution group use the _New-DynamicDistributionGroup_ cmdlet. This example creates a dynamic distribution that contains only mailbox users from the Users OU.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; New-DynamicDistributionGroup -IncludedRecipients MailboxUsers -Name MailboxUsersDDG -OrganizationalUnit Users
</pre>

This example creates a dynamic distribution group, _ITUsers_, with a custom recipient filter. It contains all mailbox users from the Users OU who are members of the _Computers_ department.

```
PS> $itUsers = New-DynamicDistributionGroup -Name ITUsers -RecipientFilter {RecipientType -eq 'UserMailbox' -and Department -eq 'Computers'} -OrganizationalUnit Users
PS> $itUsers | fl *filter

RecipientFilter     : ((((RecipientType -eq 'UserMailbox') -and (Department -eq 'Computers'))) -and (-not(Name -like 'S
                      ystemMailbox{*')) -and (-not(Name -like 'CAS_{*')) -and (-not(RecipientTypeDetailsValue -eq 'Mail
                      boxPlan')) -and (-not(RecipientTypeDetailsValue -eq 'DiscoveryMailbox')) -and (-not(RecipientType
                      DetailsValue -eq 'ArbitrationMailbox')))
LdapRecipientFilter : (&(objectClass=user)(objectCategory=person)(mailNickname=*)(msExchHomeServerName=*)(department=Co
                      mputers)(!(name=SystemMailbox{*))(!(name=CAS_{*))(!(msExchRecipientTypeDetails=16777216))(!(msExc
                      hRecipientTypeDetails=536870912))(!(msExchRecipientTypeDetails=8388608)))
```

In the next tip we will see how to list the members of the dynamic distribution group. Stay tuned.