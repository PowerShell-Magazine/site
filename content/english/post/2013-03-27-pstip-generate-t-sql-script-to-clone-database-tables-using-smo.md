---
title: '#PSTip Generate T-SQL script to clone database tables using SMO'
author: Ravikanth C
type: post
date: 2013-03-27T18:00:20+00:00
url: /2013/03/27/pstip-generate-t-sql-script-to-clone-database-tables-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier [tip][1], we looked at how we can use SMO in PowerShell to generate T-SQL scripts for cloning databases. Today, we shall see how we can clone database tables. The approach is very similar.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$server.Databases['DBName']
</pre>

Once we have the database object, we can access the &#8216;Tables&#8217; property and retrieve all the tables in the database.

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases['DBName'].Tables
</pre>

Now, if we want to clone a table in a database, we can do that using:

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases['DBName'].Tables['TableName'].Script()
</pre>

[1]: /2013/03/19/pstip-generate-t-sql-script-for-cloning-a-sql-database/