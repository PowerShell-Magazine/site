---
title: '#PSTip Validate Active Directory Credentials'
author: Deepak Dhami
type: post
date: 2015-09-04T16:00:53+00:00
url: /2015/09/04/pstip-validate-active-directory-credentials/
views:
  - 21652
post_views_count:
  - 11174
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or later.

PowerShell let&#8217;s you tap into .NET Framework and do all kind of poking. Recently, while reading up this article on [CodeProject][1]  came across the ValidateCredentials() method on the [PrincipalContext][2] class instance.

Below is how you use this nifty little trick in PowerShell to validate AD creds for a user (One can use this for local **machine** too):

```powershell
# add type to allow validating credentials
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

# Create the Domain Context for authentication
$ContextType = [System.DirectoryServices.AccountManagement.ContextType]::Domain

# We specify Negotiate as the Context option as it takes care of choosing the best authentication mechanism i.e. Kerberos or NTLM (non-domain joined machines).
$ContextOptions = [System.DirectoryServices.AccountManagement.ContextOptions]::Negotiate
```

Let’s create the instance of the PrinicipalContext class by using one of the [Constructor][3] . Note this requires a DC name to be passed. Don’t worry if you don’t know the DC name, we can easily use the $env:USERDNSDOMAIN environment variable and it takes care of it.

```powershell
$PrincipalContext = New-Object System.DirectoryServices.AccountManagement.PrincipalContext($ContextType, $env:USERDNSDOMAIN)
```


Before it is time to execute the method, let’s see the method definition

```powershell
PS> $PrincipalContext.ValidateCredentials
OverloadDefinitions
-------------------
bool ValidateCredentials(string userName, string password)
bool ValidateCredentials(string userName, string password, System.DirectoryServices.AccountManagement.ContextOptions options)
```


We use the second method definition now to validate the user credential, and we can store the user credentials in a credential object (for ease) here.

```powershell
PS> $Cred = Get-Credential
PS> $PrincipalContext.ValidateCredentials($cred.UserName, $cred.GetNetworkCredential().password, $ContextOptions)
True
```


PS># Et Voila !

[1]: http://www.codeproject.com/Articles/38344/Using-System-DirectoryServices-AccountManagement
[2]: https://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.principalcontext(v=vs.110).aspx
[3]: https://msdn.microsoft.com/en-us/library/bb298328(v=vs.110).aspx