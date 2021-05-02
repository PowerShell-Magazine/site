---
title: '#PSTip Changing SQL database AutoShrink property using SMO and PowerShell'
author: Ravikanth C
type: post
date: 2013-04-22T18:00:18+00:00
url: /2013/04/22/pstip-changing-sql-database-autoshrink-property-using-smo-and-powershell/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In today&#8217;s tip, we will see how we can use SQL SMO and PowerShell to change the AutoShrink property of a SQL database. This property specifies whether the size of the database is automatically reduced when a large amount of available space occurs. The _[AutoShrink][1]_ property takes a _Boolean_ value. So, setting it to _$true_ will enable auto-shrink of the database and _$false_ will disable it.

Let us see how we can change this.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$database = $server.databases["MyDB"]
$database.AutoShrink = $true
$database.Alter()
</pre>

[1]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.databaseoptions.autoshrink.aspx