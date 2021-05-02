---
title: '#PSTip Verify Active Directory account credentials using System.DirectoryServices.DirectoryEntry'
author: Jaap Brasser
type: post
date: 2013-05-21T18:00:00+00:00
url: /2013/05/21/pstip-verify-active-directory-account-credentials-using-system-directoryservices-directoryentry/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
The [System.DirectoryServices.AccountManagement][1] namespace provides a nice way of testing if a set of Active Directory credentials are correct (also discussed in [PSTip Validating Active Directory user credentials][2] ). Another method is utilizing the [System.DirectoryServices.DirectoryEntry][3] class to create an LDAP connection to the default domain. By default every user should be able to access this entry and therefore this can be used to verify the Active Directory credentials of a user account. The following example will show the basic workings of the class.

<pre class="brush: powershell; title: ; notranslate" title="">PS &gt; $DomainDN = ([adsi]'').distinguishedName
PS &gt; New-Object System.DirectoryServices.DirectoryEntry("LDAP://$DomainDN",'jaapbrasser','Secret01')
format-default : The following exception occurred while retrieving member "distinguishedName": "The user name or password is incorrect."
    + CategoryInfo          : NotSpecified: (:) [format-default], ExtendedTypeSystemException
    + FullyQualifiedErrorId : CatchFromBaseGetMember,Microsoft.PowerShell.Commands.FormatDefaultCommand
</pre>

Unlike System.DirectoryServices.AccountManagement, the output is not $true or $false. Instead, an error is generated if the class is provided with incorrect credentials. If the credentials are correct the returned object will contain the distinguishedName property, this property will be used to create the Boolean output.

<pre class="brush: powershell; title: ; notranslate" title="">$DomainDN = ([adsi]'').distinguishedName
$Account = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$DomainDN",'jaapbrasser','Secret01')
[bool]$Account.distinguishedName
</pre>

Now we get output similar to what the System.DirectoryServices.AccountManagement class provides. This is obviously more work to implement but you can wrap this in a function and reuse it when needed. An advantage of this class is that no additional DLLs are required for this script to function.

[1]: http://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.aspx
[2]: /2013/02/15/pstip-validating-active-directory-user-credentials/
[3]: http://msdn.microsoft.com/en-us/library/z9cddzaa.aspx