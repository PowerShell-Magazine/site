---
title: '#PSTip Backing up SQL transaction log using SMO'
author: Ravikanth C
type: post
date: 2013-04-16T18:00:11+00:00
url: /2013/04/16/pstip-backing-up-sql-transaction-log-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier tip, we looked at how we can use [SQL SMO to perform database backup][1]. In today&#8217;s tip, we shall see how we can perform transaction log backup in PowerShell.

If you have observed the code in the earlier tip, we used an enumeration called [BackupActionType][2]. We can use the same enumeration for performing a transaction log backup.

Let us see how:

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
Add-Type -AssemblyName "Microsoft.SqlServer.SMOExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"

$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$backup = New-Object Microsoft.SqlServer.Management.Smo.Backup -Property @{
   Action = [Microsoft.SqlServer.Management.Smo.BackupActionType]::Log
   BackupSetDescription = "Transaction Log backup of MyDB"
   BackupSetName = "MyDB TLog backup set"
   Database = "MyDB"
   MediaDescription = "Disk"
}

$backup.Devices.AddDevice("C:\Backup\MyDB-TLog.bak", 'File')
$backup.SqlBackup($server)
```

In the above snippet, _Action = [Microsoft.SqlServer.Management.Smo.BackupActionType]::Log_ is what defines that we want to perform a transaction log backup.

[1]: /2013/04/12/pstip-backing-up-a-sql-database-using-smo/
[2]: http://msdn.microsoft.com/en-IN/library/microsoft.sqlserver.management.smo.backupactiontype.aspx