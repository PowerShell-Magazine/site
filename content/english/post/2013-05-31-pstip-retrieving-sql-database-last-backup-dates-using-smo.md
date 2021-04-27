---
title: '#PSTip Retrieving SQL database last backup dates using SMO'
author: Ravikanth C
type: post
date: 2013-05-31T18:00:00+00:00
url: /2013/05/31/pstip-retrieving-sql-database-last-backup-dates-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

For regular reporting and auditing purposes, it is always desired to capture the SQL last backup dates. SQL SMO database properties give us this information.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$smo.Databases | Select Name, LastBackupDate, LastLogBackupDate, LastDifferentialBackupDate
</pre>

The above code gives information about all databases on the local SQL server. If you want to filter it down to a specific database, you can do that using the following command.

<pre class="brush: powershell; title: ; notranslate" title="">$smo.Databases["MyDB"] | Select LastBackupDate, LastLogBackupDate, LastDifferentialBackupDate
</pre>