---
title: '#PSTip Restoring a SQL Database using SMO'
author: Ravikanth C
type: post
date: 2013-04-18T18:00:13+00:00
url: /2013/04/18/pstip-restoring-a-sql-database-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In a couple of earlier tips on using SMO, we looked at [backing up a SQL database][1] and [transaction logs][2]. In today&#8217;s tip, we will look at how we can perform a database restore using SQL SMO and PowerShell.

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
Add-Type -AssemblyName "Microsoft.SqlServer.SMOExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"


$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName

$restore = new-object Microsoft.SqlServer.Management.Smo.Restore -Property @{
	Action = 'database'
	Database = 'MyDB'
	ReplaceDatabase = $true
	NoRecovery = $false
}

$device = New-Object -TypeName Microsoft.SqlServer.Management.Smo.BackupDeviceItem -ArgumentList "C:\Intel\MyDB.bak","File"
$restore.Devices.Add($device)
$restore.SqlRestore($Server)
```

Make a note that this code will restore database to the default SQL data file path. If we need to relocate the files to a different path during restore, we need to follow a different path. Let us save it for another day and another tip! 🙂

[1]: /2013/04/12/pstip-backing-up-a-sql-database-using-smo/
[2]: /2013/04/16/pstip-backing-up-sql-transaction-log-using-smo