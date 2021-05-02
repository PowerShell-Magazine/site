---
title: '#PSTip Verify local SAM store account credentials'
author: Jaap Brasser
type: post
date: 2013-05-22T18:00:00+00:00
url: /2013/05/22/pstip-verify-local-sam-store-account-credentials/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
PowerShell provides a nice way of testing if a set of credentials are correct. This can be done by using the [System.DirectoryServices.AccountManagement][1] namespace. Earlier this year Shay discussed how this class can be used to verify Active Directory credentials, [PSTip Validating Active Directory user credentials][2]. However it is also possible to verify local accounts. An example of how to test the local user account credentials:

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName System.DirectoryServices.AccountManagement
$DS = New-Object System.DirectoryServices.AccountManagement.PrincipalContext('machine',$env:COMPUTERNAME)
$DS.ValidateCredentials('jaapbrasser', 'Secret01') 
</pre>

The result of this code is a Boolean value, reporting back either True or False. To make this simpler I wrote an advanced function that verifies local user credentials. It is available in the Technet Script Repository: [Test-LocalCredential][3]


    function Test-LocalCredential {
        [CmdletBinding()]
    
        Param
        (
            [Parameter(Mandatory=$true)]
            [string]$UserName,
            [string]$ComputerName = $env:COMPUTERNAME,
            [Parameter(Mandatory=$true)]
            [string]$Password
        ) 
    
        Add-Type -AssemblyName System.DirectoryServices.AccountManagement
        $DS = New-Object System.DirectoryServices.AccountManagement.PrincipalContext('machine',$ComputerName)
        $DS.ValidateCredentials($UserName, $Password)
    }
This function can be called  as shown in the next example:

```
PS> Test-LocalCredential -UserName jaapbrasser -Password Secret01
True
```


[1]: http://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.aspx
[2]: /2013/02/15/pstip-validating-active-directory-user-credentials/
[3]: http://gallery.technet.microsoft.com/Verify-the-Local-User-1e365545