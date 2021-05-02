---
title: '#PSTip Validate is a database is available on a SQL instance'
author: Hemanth C Damecharla
type: post
date: 2013-08-21T18:00:37+00:00
url: /2013/08/21/pstip-validate-is-a-database-is-available-on-a-sql-instance/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

Sometimes we need to know if a particular DB is available on a given instance before going ahead and making any configuration changes. The below code snippet allows you to quickly check if a DB is available on a particular SQL Server instance.

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME

if($server.Databases["DatabaseName"]) {
    $true
} else {
    $false
}
```

One thing to consider is that the state of the database will not be considered here. For example, if the database is &#8216;Offline&#8217;; the conditional will still return &#8216;True&#8217; as the DB is available in the catalog. So, if we want to check if the DB is available and also accessible then we can use the following method. It returns a boolean value based on the database accessibility.

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases["DatabaseName"].IsAccessible
</pre>