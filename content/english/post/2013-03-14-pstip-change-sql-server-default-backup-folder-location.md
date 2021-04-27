---
title: '#PSTip Change SQL Server default backup folder location'
author: Ravikanth C
type: post
date: 2013-03-14T18:00:29+00:00
url: /2013/03/14/pstip-change-sql-server-default-backup-folder-location/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

If you have ever backed up a SQL Server database, you will know that the default backup location is set to the Program Files folder where SQL is installed. For example, on a system with SQL Server 2008 R2, it is set to C:\Program Files\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\Backup. This may not always be an ideal location to store database backups.

So, how do we change this using PowerShell? We can either use Windows Registry to change this path or SQL Management Objects (SMO) to do this. We shall see how we can use SMO in this tip.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server($env:ComputerName)
$server.Properties["BackupDirectory"].Value = "K:\Backup"
$server.Alter()
</pre>

Make a note that the assembly version specified in _Add-Type_ command above is SQL Server 2008 R2 version. If you want to use this code snippet for SQL Server 2012, you&#8217;d have to find the right SMO version.