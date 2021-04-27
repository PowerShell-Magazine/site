---
title: '#PSTip Backing up a SQL database using SMO'
author: Ravikanth C
type: post
date: 2013-04-12T18:00:26+00:00
url: /2013/04/12/pstip-backing-up-a-sql-database-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In today&#8217;s tip, we shall see how we can use SQL Management Objects (SMO) in PowerShell to perform a SQL database backup. The [Microsoft.SqlServer.Management.Smo.Backup][1] class can be used to achieve this.

Let us see how:

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
add-type -AssemblyName "Microsoft.SqlServer.SMOExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
</pre>

Once we have the server object, we can perform backup by running the following code snippet.

<pre class="brush: powershell; title: ; notranslate" title="">$backup = New-Object Microsoft.SqlServer.Management.Smo.Backup -Property @{
   Action = [Microsoft.SqlServer.Management.Smo.BackupActionType]::Database
   BackupSetDescription = "Full backup of MyDB"
   BackupSetName = "MyDB backup set"
   Database = "MyDB"
   MediaDescription = "Disk"
}
$backup.Devices.AddDevice("C:\Backup\MyDB.bak", 'File')
$backup.SqlBackup($server)
</pre>

[1]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.backup.aspx