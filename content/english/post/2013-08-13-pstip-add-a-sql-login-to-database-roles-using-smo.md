---
title: '#PSTip Add a SQL login to database roles using SMO'
author: Ravikanth C
type: post
date: 2013-08-13T18:00:54+00:00
url: /2013/08/13/pstip-add-a-sql-login-to-database-roles-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

We can use SQL Server SMO object to add a SQL login to the database roles. For example, roles such as dbcreator, sysadmin, etc.

Let us see how we can do it using PowerShell and SMO.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$smo.Logins["Domain\SQLUser"].AddToRoles('sysadmin')
</pre>