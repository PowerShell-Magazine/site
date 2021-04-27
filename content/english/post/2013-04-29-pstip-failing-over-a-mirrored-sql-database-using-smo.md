---
title: '#PSTip Failing over a mirrored SQL database using SMO'
author: Ravikanth C
type: post
date: 2013-04-29T06:00:00+00:00
url: /2013/04/29/pstip-failing-over-a-mirrored-sql-database-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

After I configure SQL mirroring, I usually perform some checks to ensure that the overall mirroring configuration is working fine. This is an essential step to ensure continuous availability of the databases. Checking database failover is one of the items on this checklist. For this purpose, I use the following code to manually failover a single database to the partner SQL instance.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91";
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$smo.Databases['MyDB'].ChangeMirroringState('Failover')
</pre>

Make a note that you need to first verify whether the database is mirrored or not before you can use ChangeMirroringState() method. We can do this by using the IsMirroringEnabled property of a database.

<pre class="brush: powershell; title: ; notranslate" title="">$smo.Databases['MyDB'].IsMirroringEnabled
</pre>