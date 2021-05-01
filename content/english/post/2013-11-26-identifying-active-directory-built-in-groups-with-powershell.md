---
title: Identifying Active Directory built-in groups with PowerShell
author: Shay Levy
type: post
date: 2013-11-26T17:00:26+00:00
url: /2013/11/26/identifying-active-directory-built-in-groups-with-powershell/
categories:
  - How To
  - Active Directory
tags:
  - Active Directory
  - How To

---
Built-in groups are predefined security groups, defined with domain local scope, that are created automatically when you create an Active Directory domain. You can use these groups to control access to shared resources and delegate specific domain-wide administrative roles. Built-in groups are located under the Builtin container. One way to list all built-in groups is to get all groups from the Builtin container.

```
PS> Get-ADGroup -SearchBase 'CN=Builtin,DC=domain,DC=com' -Filter * |
Format-Table Name,GroupScope,GroupCategory,SID

Name                                GroupScope GroupCategory SID
----                                ---------- ------------- ---
Network Configuration Operators    DomainLocal      Security S-1-5-32-556
Incoming Forest Trust Builders     DomainLocal      Security S-1-5-32-557
Distributed COM Users              DomainLocal      Security S-1-5-32-562
Guests                             DomainLocal      Security S-1-5-32-546
Performance Log Users              DomainLocal      Security S-1-5-32-559
Performance Monitor Users          DomainLocal      Security S-1-5-32-558
Pre-Windows 2000 Compatible Access DomainLocal      Security S-1-5-32-554
Terminal Server License Servers    DomainLocal      Security S-1-5-32-561
Users                              DomainLocal      Security S-1-5-32-545
Windows Authorization Access Group DomainLocal      Security S-1-5-32-560
Remote Desktop Users               DomainLocal      Security S-1-5-32-555
Replicator                         DomainLocal      Security S-1-5-32-552
Print Operators                    DomainLocal      Security S-1-5-32-550
Administrators                     DomainLocal      Security S-1-5-32-544
Backup Operators                   DomainLocal      Security S-1-5-32-551
Account Operators                  DomainLocal      Security S-1-5-32-548
Server Operators                   DomainLocal      Security S-1-5-32-549
IIS_IUSRS                          DomainLocal      Security S-1-5-32-568
Cryptographic Operators            DomainLocal      Security S-1-5-32-569
Event Log Readers                  DomainLocal      Security S-1-5-32-573
Certificate Service DCOM Access    DomainLocal      Security S-1-5-32-574
Power Users                             Global      Security S-1-5-21-33041744-1045339714-1466953526-11229
```

Although groups in the Builtin container cannot be moved to another location, other group types are found in that location. From the above output you can see that the _&#8216;Power Users&#8217;_  group is not a &#8216;pure&#8217; built-in group. Another way to list built-in groups is to filter them by a portion of their SID value:

```
PS> Get-ADGroup -ResultSetSize $null -Filter * -Properties GroupType |
Where-Object {$_.SID -like "S-1-5-32-*"} |
Format-Table Name,GroupScope,GroupCategory,GroupType,SID

Name                                GroupScope GroupCategory   GroupType SID
----                                ---------- -------------   --------- ---
Network Configuration Operators    DomainLocal      Security -2147483643 S-1-5-32-556
Incoming Forest Trust Builders     DomainLocal      Security -2147483643 S-1-5-32-557
Distributed COM Users              DomainLocal      Security -2147483643 S-1-5-32-562
Guests                             DomainLocal      Security -2147483643 S-1-5-32-546
Performance Log Users              DomainLocal      Security -2147483643 S-1-5-32-559
Performance Monitor Users          DomainLocal      Security -2147483643 S-1-5-32-558
Pre-Windows 2000 Compatible Access DomainLocal      Security -2147483643 S-1-5-32-554
Replicator                         DomainLocal      Security -2147483643 S-1-5-32-552
Print Operators                    DomainLocal      Security -2147483643 S-1-5-32-550
Administrators                     DomainLocal      Security -2147483643 S-1-5-32-544
Backup Operators                   DomainLocal      Security -2147483643 S-1-5-32-551
Account Operators                  DomainLocal      Security -2147483643 S-1-5-32-548
Server Operators                   DomainLocal      Security -2147483643 S-1-5-32-549
Windows Authorization Access Group DomainLocal      Security -2147483643 S-1-5-32-560
Terminal Server License Servers    DomainLocal      Security -2147483643 S-1-5-32-561
Users                              DomainLocal      Security -2147483643 S-1-5-32-545
Remote Desktop Users               DomainLocal      Security -2147483643 S-1-5-32-555
Cryptographic Operators            DomainLocal      Security -2147483643 S-1-5-32-569
IIS_IUSRS                          DomainLocal      Security -2147483643 S-1-5-32-568
Event Log Readers                  DomainLocal      Security -2147483643 S-1-5-32-573
Certificate Service DCOM Access    DomainLocal      Security -2147483643 S-1-5-32-574
```

This method is not the most efficient one because it first gets **ALL** groups from the directory (depending on the size of your environment that can take some time to finish) and only then filters them by their SID. We could try and filter them using the _Filter_ parameter (e.g _Get-ADGroup -Filter {SID -like &#8220;S-1-5-32-*&#8221;}_) but filtering by SID on the server is not supported and doesn&#8217;t yield a result.

So, how can we get all the groups in an efficient manner and make sure we get built-in groups only? First, let&#8217;s find out what makes a group a built-in group. We will investigate the groupType value of the groups (see output above). The groupType value is a [combination of flags][1] that determine the type of the group.

| Symbolic name                  | Value      |
| ------------------------------ | ---------- |
| GROUP_TYPE_BUILTIN_LOCAL_GROUP | 0x00000001 |
| GROUP_TYPE_ACCOUNT_GROUP       | 0x00000002 |
| GROUP_TYPE_RESOURCE_GROUP      | 0x00000004 |
| GROUP_TYPE_UNIVERSAL_GROUP     | 0x00000008 |
| GROUP_TYPE_APP_BASIC_GROUP     | 0x00000010 |
| GROUP_TYPE_APP_QUERY_GROUP     | 0x00000020 |
| GROUP_TYPE_SECURITY_ENABLED    | 0x80000000 |

The groupType AD property returns a numeric value but using _ADSI Edit (adsiedit.msc)_ you can get a visual representation of the flags.

![](/images/groupType.png)

As you can see the Administrators group type is marked with three attributes: _ GROUP\_TYPE\_BUILTIN\_LOCAL\_GROUP, GROUP\_TYPE\_RESOURCE_GROUP_, and _GROUP\_TYPE\_SECURITY_ENABLED_. These attributes translates to the flags in the figure above.

```
PS> $GROUP_TYPE_BUILTIN_LOCAL_GROUP = 0x00000001
PS> $GROUP_TYPE_RESOURCE_GROUP = 0x00000004
PS> $GROUP_TYPE_SECURITY_ENABLED = 0x80000000
PS> $groupType = $GROUP_TYPE_BUILTIN_LOCAL_GROUP + $GROUP_TYPE_RESOURCE_GROUP + $GROUP_TYPE_SECURITY_ENABLED
PS> $groupType
-2147483643

PS> '0x{0:x}' -f $groupType
0x80000005
```

In the case of built-in groups, they all have the same attributes applied so you could list them using the _Filter_ parameter with the following command:

```
Get-ADGroup -Filter {GroupType -eq -2147483643}
```


That&#8217;s much more efficient than using a _Where-Object_ command but it gets us all groups with all of those attributes defined together. If you want to test if a group has the  _BUILTIN\_LOCAL\_GROUP_ flag turned on, you can do that using the _bitwise AND (-bAnd)_ operator.

<pre class="brush: powershell; title: ; notranslate" title="">Get-ADGroup -Filter {GroupType -band 1}
</pre>

[1]: http://msdn.microsoft.com/en-us/library/cc223142.aspx