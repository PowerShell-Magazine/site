---
title: '#PSTip List SQL database mirroring partner using SMO'
author: Ravikanth C
type: post
date: 2013-05-14T18:00:00+00:00
url: /2013/05/14/pstip-list-sql-database-mirroring-partner-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier tip, we looked at how to verify if a SQL database is mirrored. Continuing this series on SQL SMO tips series, let us look at how we can list the SQL database mirroring partner information. I find this information quite helpful when managing the SQL Server mirroring. The database class in SQL SMO has a property called [MirroringPartner][1]. This provides us the information of the Database Engine instance that is the partner server for database mirroring.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$smo.Databases['MyDB'].MirroringPartner
</pre>

The above code snippet returns the mirroring partner FQDN along with the TCP port number. In case you need only the name of the SQL Server instance hosting the mirroring partner, you can use the following code.

<pre class="brush: powershell; title: ; notranslate" title="">$smo.Databases['MyDB'].MirroringPartnerInstance
</pre>

[1]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.database.mirroringpartner.aspx