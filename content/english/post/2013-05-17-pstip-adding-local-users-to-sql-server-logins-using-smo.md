---
title: '#PSTip Adding local users to SQL Server Logins using SMO'
author: Ravikanth C
type: post
date: 2013-05-17T18:00:07+00:00
url: /2013/05/17/pstip-adding-local-users-to-sql-server-logins-using-smo/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

There was a question on StackOverflow about [adding local users to SQL Server logins][1]. I provided an answer to that and realized from a user&#8217;s comment that the SMO method to create a SQL login works only in a specified manner.

So, in this tip, I will show you the SMO way of adding local users to SQL Server logins.

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
Add-Type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"

$conn = New-Object Microsoft.SqlServer.Management.Common.ServerConnection -ArgumentList $env:ComputerName
$conn.applicationName = "PowerShell SMO"
$conn.ServerInstance = ".\SQLEXPRESS"
$conn.StatementTimeout = 0
$conn.Connect()
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server -ArgumentList $conn
$SqlUser = New-Object -TypeName Microsoft.SqlServer.Management.Smo.Login -ArgumentList $smo,"${env:ComputerName}\JohnDoe"
$SqlUser.LoginType = 'WindowsUser'
$sqlUser.PasswordPolicyEnforced = $false
$SqlUser.Create()
```

In the above code snippet, observe the way we specified (line 10) the username for the login. It needs to be prefixed with the computer name and not just &#8216;localhost&#8217;.

[1]: http://stackoverflow.com/questions/16610379/add-windows-user-to-local-sql-server-with-powershell