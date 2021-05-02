---
title: '#PSTip Extending the default output property set for Active Directory objects'
author: Shay Levy
type: post
date: 2013-04-30T18:00:00+00:00
url: /2013/04/30/pstip-extending-the-default-output-property-set-for-active-directory-objects/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

When you get user objects from Active Directory using the [Active Directory module cmdlets][1], you get a &#8216;thin&#8217; output object that includes just a few properties.

```
PS> Get-ADUser shay

DistinguishedName : CN=shay,CN=Users,DC=domain,DC=com
Enabled           : True
GivenName         : Shay
Name              : Shay
ObjectClass       : user
ObjectGUID        : 3a596c36-c876-43a5-a0af-55865cd0cb1d
SamAccountName    : Shay
SID               : S-1-5-21-17886608-6971410453-5352618669-29132
Surname           : Shay
UserPrincipalName : shay@domain.com
```

This can boost your performance when you issue a query that returns many objects. Sometimes, however, you&#8217;ll want to return more data contained in properties that are not included in the default set. You can do that by specifying a comma separated list of property names in the _Properties_ parameter. Wildcards are also permitted, and specifying an asterisk _&#8216;*&#8217;_ will get all properties. The following command adds the _Office,Department,mail, and GivenName_ properties to the output object.

```
PS> Get-ADUser shay -Properties Office,Department,mail,GivenName,sn

Department        : Computers
DistinguishedName : CN=shay,CN=Users,DC=domain,DC=com
Enabled           : True
GivenName         : Shay
mail              : shay@domain.com
Name              : Shay
ObjectClass       : user
ObjectGUID        : 3a596c36-c876-43a5-a0af-55865cd0cb1d
Office            : Computers
SamAccountName    : Shay
SID               : S-1-5-21-17886608-6971410453-5352618669-29132
Surname           : Levy
UserPrincipalName : shay@domain.com
```

Great but most of the time this is how you&#8217;ll want your result to look like by default and not having to explicitly declare it each time you run the command.

<p style="direction: ltr;">
  There is no cmdlet that let&#8217;s configure this, but luckily you can control it with a new feature of PowerShell 3.0: <a href="http://technet.microsoft.com/en-us/library/hh847819.aspx">Parameters default values</a>. This feature allows us to specify custom default values for any cmdlet or advanced function. Let&#8217;s see how we can use it with the <em>Get-ADUser</em> command:
</p>
```
PS> $PSDefaultParameterValues=@{'Get-ADUser:Properties' = 'Office','Department','mail','GivenName','sn'}
PS> Get-ADUser shay

Department        : Computers
DistinguishedName : CN=shay,CN=Users,DC=domain,DC=com
Enabled           : True
GivenName         : Shay
mail              : shay@domain.com
Name              : Shay
ObjectClass       : user
ObjectGUID        : 3a596c36-c876-43a5-a0af-55865cd0cb1d
Office            : Computers
SamAccountName    : Shay
SID               : S-1-5-21-17886608-6971410453-5352618669-29132
Surname           : Levy
UserPrincipalName : shay@domain.com
```

Awesome! All that&#8217;s left to do is add the _$PSDefaultParameterValues_ command to your PowerShell profile so that each time you run _Get-ADUser,_ the output object will have an extended set of attributes. In a similar manner, you can create default sets for computer objects and so on.

[1]: http://technet.microsoft.com/en-us/library/ee617195.aspx