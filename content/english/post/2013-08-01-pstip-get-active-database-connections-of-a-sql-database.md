---
title: '#PSTip Get Active Database Connections of a SQL Database'
author: Ravikanth C
type: post
date: 2013-08-01T18:00:56+00:00
url: /2013/08/01/pstip-get-active-database-connections-of-a-sql-database/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

As a part of SQL SMO automation, when we want to detach or delete databases, we might want to first drop all active database connections. But how do we even know whether there are any active connections to the database or not?

We can use the [Server SMO][1] and the [GetActiveDBConnectionCount()][2] method for retrieving the connection count.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$smo.GetActiveDBConnectionCount('MyDB')
</pre>

This code snippet returns an integer number indicating the active DB connections.

[1]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.server.aspx
[2]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.server.getactivedbconnectioncount.aspx