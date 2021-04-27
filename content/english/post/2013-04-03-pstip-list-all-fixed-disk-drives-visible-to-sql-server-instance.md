---
title: '#PSTip List all fixed disk drives visible to SQL Server instance'
author: Ravikanth C
type: post
date: 2013-04-03T18:00:54+00:00
url: /2013/04/03/pstip-list-all-fixed-disk-drives-visible-to-sql-server-instance/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

When automating SQL server database creation in PowerShell, we may want to give the end user an option to select from a list of available fixed disk drives and let the user select the drive that should be used for database file creation, etc. It is preferred here to see what drives can be seen by the SQL server instance. We can do this using the [EnumAvailableMedia()][1] method in SQL SMO. A variant of this method takes [MediaType][2] as an argument. Within this enumeration, a value &#8216;2&#8217; represents fixed disk.

Let us see how to use this in PowerShell:

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
$server.EnumAvailableMedia(2)
</pre>

[1]: http://msdn.microsoft.com/en-us/library/ms210215.aspx
[2]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.mediatypes.aspx