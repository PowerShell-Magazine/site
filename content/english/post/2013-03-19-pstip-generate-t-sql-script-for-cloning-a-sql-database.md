---
title: '#PSTip Generate T-SQL script for cloning a SQL database'
author: Ravikanth C
type: post
date: 2013-03-19T18:00:10+00:00
url: /2013/03/19/pstip-generate-t-sql-script-for-cloning-a-sql-database/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

As database administrator, you may want to create a database on development or test servers with similar settings as on the production server. This is usually done using T-SQL. So, how is this related to PowerShell?

We can use SQL SMO objects in PowerShell to generate this T-SQL script!

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
</pre>

Once we have the SQL Server object, we can use the Databases property to list all databases on the server.

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases
</pre>

The database object has the Script() method which can be used to generate the T-SQL script!

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases['DBName'].Script()
</pre>