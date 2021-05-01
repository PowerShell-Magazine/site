---
title: '#PSTip Drop all Active Connections of SQL Database'
author: Ravikanth C
type: post
date: 2013-08-02T18:00:04+00:00
url: /2013/08/02/pstip-drop-all-active-connections-of-sql-database/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier tip, we looked at how we can [retrieve the active connection count for a SQL database][1]. In today&#8217;s tip we will look at how we can drop all the active connections before we can perform a detach database operation.

For this purpose, we will use the [KillAllProcesses()][2] method of the [Server SMO][3].

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$smo.KillAllProcesses('MyDB')
</pre>

This code snippet will help you drop all active database connections of a given SQL database.

[1]: http://104.131.21.239/2013/07/31/pstip-get-active-database-connections-of-a-sql-database
[2]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.server.killallprocesses.aspx
[3]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.server.aspx