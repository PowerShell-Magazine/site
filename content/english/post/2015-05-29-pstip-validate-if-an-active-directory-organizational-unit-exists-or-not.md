---
title: '#PSTip Validate if an Active Directory Organizational Unit exists or not'
author: Ravikanth C
type: post
date: 2015-05-29T18:00:08+00:00
url: /2015/05/29/pstip-validate-if-an-active-directory-organizational-unit-exists-or-not/
views:
  - 8306
post_views_count:
  - 1931
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
For some recent scripting work, I needed to find if an AD OU existed or not from a computer that is not domain-joined. I knew that the [System.DirectoryServices.DirectoryEntry][1] class can help but I wasn&#8217;t sure how to specify the path.

When I finally figured out, here is how I provided path.

```powershell
New-Object System.DirectoryServices.DirectoryEntry("LDAP://DomainFQDN/OU=OUName,DC=DOMAIN,DC=Suffix",'UserName','Password')
```


You need to replace the _DomainFQDN_ in the above code sample to something like Contoso.com, _OUName_ with the name of the organizational unit you want to find, _Domain_ with the NETBIOS name like Contoso and finally the Suffix with COM.

```powershell
New-Object System.DirectoryServices.DirectoryEntry("LDAP://Contoso.com/OU=TestUnit,DC=Contoso,DC=com",'administrator','MyP@ssw0rd')
```


If the object creation succeeds, it will return the object that will have the _distinguishedname_ property set to the path of the OU.

This, of course, assumes that the credentials specified are correct. If the credentials are invalid, you will see an error that the username or password may be wrong.

[1]: https://msdn.microsoft.com/en-us/library/system.directoryservices.directoryentry%28v=vs.110%29.aspx