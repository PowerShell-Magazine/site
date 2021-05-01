---
title: '#PSTip Finding all roles a particular login is mapped to in all databases'
author: Hemanth C Damecharla
type: post
date: 2013-08-20T18:00:16+00:00
url: /2013/08/20/pstip-finding-all-roles-a-particular-login-is-mapped-to-in-all-databases/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In a [previous tip][1], we checked out how to get the names of all databases to which a login is mapped and the username for that login in each database. Once, we have this information, we can enumerate the roles that are assigned to the user mapped to our login using [EnumRoles()][2] method.

We will use the _Test-SQLLogin_ function described in an [earlier tip][3].

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$login = 'TestUser' #SQLlogin or a windows login.

if((Test-SQLLogin -SqlLogin $login))
{
    $server.Logins["$login"].EnumDatabaseMappings() |
    Select DBName, UserName, @{Name="AssignedRoles"; Expression={@($server.Databases[$_.DBName].Users[$_.UserName].EnumRoles())}}
}

DBName UserName AssignedRoles
------ -------- -------------
master TestUser {db_datareader, db_datawriter}
msdb   TestUser {db_backupoperator, db_datareader, db_datawriter}
```

[1]: /2013/08/19/pstip-finding-all-databases-to-which-a-login-is-mapped/
[2]: http://technet.microsoft.com/en-in/library/microsoft.sqlserver.management.smo.databaserole.enumroles.aspx
[3]: /2013/08/14/pstip-validate-if-a-sql-login-exists-using-powershell/