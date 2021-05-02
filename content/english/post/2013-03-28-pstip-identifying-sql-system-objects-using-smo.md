---
title: '#PSTip Identifying SQL system objects using SMO'
author: Ravikanth C
type: post
date: 2013-03-28T18:00:21+00:00
url: /2013/03/28/pstip-identifying-sql-system-objects-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

A SQL Server instance, by default, deploys a few system databases such as Temp DB, MSDB, Master, and Model DB. Other databases that we create are called user databases. When performing automated actions, we should ensure that we don&#8217;t modify any settings of these system databases or any system objects for that matter.

So, how do we identify what are the system objects?

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$server.Databases | Select Name, IsSystemObject
</pre>

We can use the similar approach for finding tables that are system objects.

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases['DBName'].Tables | Select Name, IsSystemObject
</pre>