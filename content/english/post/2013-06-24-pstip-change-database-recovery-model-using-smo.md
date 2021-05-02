---
title: '#PSTip Change database recovery model using SMO'
author: Ravikanth C
type: post
date: 2013-06-24T18:00:42+00:00
url: /2013/06/24/pstip-change-database-recovery-model-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - SQL
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Database mirroring requires that the database be in Full recovery model as a prerequisite to configure mirroring. In this post, we shall see how we can retrieve the recovery model setting for a database and then set it to Full recovery model, as required.

We can get the recovery model setting of a given SQL database by looking at the Database object properties.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
Add-Type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$conn = New-Object Microsoft.SqlServer.Management.Common.ServerConnection -ArgumentList $env:ComputerName
$conn.applicationName = "PowerShell SMO"
$conn.ServerInstance = ".\SQLEXPRESS"
$conn.StatementTimeout = 0
$conn.Connect()
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server -ArgumentList $conn
$smo.Databases["MyDB"] | Select Name, RecoveryModel
</pre>

This property can be modified to set a database to Full recovery model. Assuming that MyDB is in Simple recovery model, we can change the property.

<pre class="brush: powershell; title: ; notranslate" title="">$smo.Databases["MyDB"].RecoveryModel = "Full"
$smo.Databases["MyDB"].Alter()
</pre>