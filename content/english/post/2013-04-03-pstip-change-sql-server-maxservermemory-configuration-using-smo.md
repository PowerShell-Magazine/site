---
title: '#PSTip Change SQL Server MaxServerMemory configuration using SMO'
author: Ravikanth C
type: post
date: 2013-04-03T18:00:17+00:00
url: /2013/04/03/pstip-change-sql-server-maxservermemory-configuration-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

As database administrators, we might want to configure SQL MaxServerMemory setting to ensure the SQL service does not occupy all available physical memory. This setting can be changed using SMO and PowerShell.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$server.Configuration.MaxServerMemory.ConfigValue = 16384
$server.Configuration.Alter()
</pre>

Make a note that the value of MaxServerMemory is in megabytes (MB).