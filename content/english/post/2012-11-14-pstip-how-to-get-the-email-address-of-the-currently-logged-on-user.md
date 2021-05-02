---
title: '#PSTip How to get the email address of the currently logged-on user'
author: Shay Levy
type: post
date: 2012-11-14T19:00:05+00:00
url: /2012/11/14/pstip-how-to-get-the-email-address-of-the-currently-logged-on-user/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
We&#8217;ll use the [adsisearcher] type accelerator. The [adsisearcher] type is just a shortcut to the System.DirectoryServices.DirectorySearcher .NET class.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [adsisearcher].FullName
System.DirectoryServices.DirectorySearcher
</pre>

We can use the accelerator to create a DirectorySearcher instance by supplying a LDAP filter. We can refine the search by changing other properties of the search object.

```
PS> $searcher = [adsisearcher]"(objectClass=user)"
PS> $searcher

CacheResults             : True
ClientTimeout            : -00:00:01
PropertyNamesOnly        : False
Filter                   : (objectClass=user)
PageSize                 : 0
PropertiesToLoad         : {}
ReferralChasing          : External
SearchScope              : Subtree
ServerPageTimeLimit      : -00:00:01
ServerTimeLimit          : -00:00:01
SizeLimit                : 0
SearchRoot               :
Sort                     : System.DirectoryServices.SortOption
Asynchronous             : False
Tombstone                : False
AttributeScopeQuery      :
DerefAlias               : Never
SecurityMasks            : None
ExtendedDN               : None
DirectorySynchronization :
VirtualListView          :
Site                     :
Container                :
```

For our purpose&#8211;finding the currently logged-on user&#8217;s email address&#8211;we want to pass the user name as the value for the SamAccountName property and ask for one result only. We will use the $env:USERNAME environment variable, and then query the mail property.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $searcher = [adsisearcher]"(samaccountname=$env:USERNAME)"
PS&gt; $searcher.FindOne().Properties.mail
</pre>