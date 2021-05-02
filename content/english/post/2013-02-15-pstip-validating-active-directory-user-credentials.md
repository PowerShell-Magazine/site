---
title: '#PSTip Validating Active Directory user credentials'
author: Shay Levy
type: post
date: 2013-02-15T19:01:15+00:00
url: /2013/02/15/pstip-validating-active-directory-user-credentials/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

There are times when you need to validate the credentials of an Active Directory user account. A typical scenario is when you have a service object that no one remembers its credentials and you don&#8217;t want to reset it before you make sure you tried all the passwords the object may has.

One option would be to try and log on to the server using those credentials. However, you can&#8217;t use that if you want to automate the process. In this case you&#8217;d want to check the _[PrincipalContext.ValidateCredentials][1]_ method.

The _ValidateCredentials_ method returns a Boolean value that specifies whether the specified username and password are valid. To use that method we first need to load the _System.DirectoryServices.AccountManagement assembly_ (part of .NET 3.5). We create a _ContextType_ object, pass it together with the user domain name to the _PrincipalContext_ object, and then invoke the method.

```
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

$UserName=$env:USERNAME
$Password='P@ssword'
$Domain = $env:USERDOMAIN

$ct = [System.DirectoryServices.AccountManagement.ContextType]::Domain
$pc = New-Object System.DirectoryServices.AccountManagement.PrincipalContext $ct,$Domain
$pc.ValidateCredentials($UserName,$Password)
```

[1]: http://msdn.microsoft.com/en-us/library/bb154889.aspx