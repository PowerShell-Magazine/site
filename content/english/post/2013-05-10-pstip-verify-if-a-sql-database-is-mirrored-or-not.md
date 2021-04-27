---
title: '#PSTip Verify if a SQL database is mirrored or not'
author: Ravikanth C
type: post
date: 2013-05-10T18:00:00+00:00
url: /2013/05/10/pstip-verify-if-a-sql-database-is-mirrored-or-not/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

When using SQL SMO to work with mirroring configuration, it is essential to verify if the database is mirrored, or not, before performing any other operations. In today&#8217;s tip, we shall see how to use SQL SMO in PowerShell to achieve this.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$smo.Databases['MyDB'].IsMirroringEnabled
</pre>

The [IsMirroringEnabled][1] property returns a boolean value that specifies whether mirroring is enabled on the database. If True, the database has mirroring enabled. Otherwise, False is returned.

[1]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.database.ismirroringenabled.aspx