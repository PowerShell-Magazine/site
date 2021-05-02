---
title: '#PSTip Validate if a user exists in Active Directory'
author: Ravikanth C
type: post
date: 2013-08-15T18:00:23+00:00
url: /2013/08/15/pstip-validate-if-a-user-exists-in-active-directory/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

[adsisearcher] is a PowerShell type adapter for [DirectorySearcher][1] .NET class. We can use this type adapter to perform queries against Active Directory Domain Services.

In this tip, we shall see how we can validate if a given user exists in AD or not. This is how we do it.

<pre class="brush: powershell; title: ; notranslate" title="">[bool]([adsisearcher]"samaccountname=username").FindOne()
</pre>

This snippet returns True or False based on if the user exists or not.

[1]: http://msdn.microsoft.com/en-us/library/system.directoryservices.directorysearcher.aspx