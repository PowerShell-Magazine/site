---
title: '#PSTip Using CIM cmdlets to find the user group membership'
author: Ravikanth C
type: post
date: 2013-02-08T19:00:07+00:00
url: /2013/02/08/pstip-using-cim-cmdlets-to-find-the-user-group-membership/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Recently, on Windows Server 2012, I was trying to determine, both locally and remotely, if a user belongs to a specific group. For example, when I am running Hyper-V automation scripts, I need to determine if the currently logged-in user is part of Hyper-V administrators group or not.

In PowerShell 3.0, we can use CIM cmlets and associations to find this easily.

```
Get-CimInstance -Filter "Name='Jeff'" -ClassName Win32_UserAccount |
Get-CimAssociatedInstance -Association Win32_GroupUser |
Select-Object Name
```


Simple!

![](/images/group1.png)

Now, to find the group memberships of the locally logged-in user on a remote machine, we just need to run:

```
Get-CimInstance -ClassName Win32_UserAccount -Filter "Name='$env:UserName'" -ComputerName Server-01 | 
Get-CimAssociatedInstance -Association Win32_GroupUser |
Select-Object Name
```

