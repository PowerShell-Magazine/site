---
title: '#PSTip Retrieve group membership of an Active Directory group recursively'
author: Shay Levy
type: post
date: 2013-05-23T18:00:00+00:00
url: /2013/05/23/pstip-retrieve-group-membership-of-an-active-directory-group-recursively/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

When you need to manage Active Directory, the [Active Directory PowerShell module][1] is the first admin choice as it provides many cmdlets for administering and interfacing with various AD objects. For example, to get the members of an AD group you&#8217;d use the _Get-ADGroupMember_ cmdlet. But what do you do when the AD module is not available in your environment?

Starting with .NET 3.5 you can load the [_System.DirectoryServices.AccountManagement_][2] assembly and use its classes and types to get the members of the group. With the following snippet you can get all members of an AD group, including nested members.

```
$Recurse = $true
$GroupName = 'Domain Admins'
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

# use the 'Machine' ContextType if you want to retrieve local group members
# for possible values of the numeration, visit
# http://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.contexttype.aspx

$ct = [System.DirectoryServices.AccountManagement.ContextType]::Domain
$group = [System.DirectoryServices.AccountManagement.GroupPrincipal]::FindByIdentity($ct,$GroupName)
$group.GetMembers($Recurse)
```

One important thing to keep in mind, the returned collection does not contain group objects when the recursive flag is set to true.

[1]: http://technet.microsoft.com/en-us/library/ee617195.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.aspx