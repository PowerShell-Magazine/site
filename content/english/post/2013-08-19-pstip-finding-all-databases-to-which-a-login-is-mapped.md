---
title: '#PSTip Finding all databases to which a login is mapped'
author: Hemanth C Damecharla
type: post
date: 2013-08-19T18:00:12+00:00
url: /2013/08/19/pstip-finding-all-databases-to-which-a-login-is-mapped/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

One of the projects on which I work requires a validation step after each deployment. We need to validate if the logins have been mapped to all required roles in each of the application databases via users created for the login.

In todays tip, we will see how we can find all the databases to which a login is mapped. The login object in SMO has a method called [EnumDatabaseMappings()][1] which enumerates the login account mappings to databases and database users. We will use the _Test-SQLLogin_ function described in an [earlier tip][2].

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$login = 'TestUser'


if((Test-SQLLogin -SqlLogin $login))
{
    $server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
    $server.Logins["$login"].EnumDatabaseMappings()| Select DBName, UserName
}

DBName UserName
------ --------
master TestUser
msdb TestUser
```

[1]: http://technet.microsoft.com/en-in/library/microsoft.sqlserver.management.smo.login.enumdatabasemappings.aspx
[2]: /2013/08/14/pstip-validate-if-a-sql-login-exists-using-powershell/