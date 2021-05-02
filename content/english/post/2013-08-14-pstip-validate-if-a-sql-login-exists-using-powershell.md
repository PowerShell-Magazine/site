---
title: '#PSTip Validate if a SQL login exists using PowerShell'
author: Ravikanth C
type: post
date: 2013-08-14T18:00:03+00:00
url: /2013/08/14/pstip-validate-if-a-sql-login-exists-using-powershell/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 3.0 or above.

Before performing tasks like adding roles to a SQL login, it is desired to validate the existence of SQL login. This can be done using the Server SMO.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
if (($smo.logins).Name -contains 'MyServerLogin') {
    "SQL login exists"
} else {
    "SQL login does not exist"
}
</pre>

We can wrap this into a reusable function.

<pre class="brush: powershell; title: ; notranslate" title="">Function Test-SQLLogin {
    param (
        [string]$SqlLogin
    )
    Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    $smo = New-Object Microsoft.SqlServer.Management.Smo.Server $env:ComputerName
    if (($smo.logins).Name -contains $SqlLogin) {
       $true
    } else {
       $false
    }
}
</pre>
<pre class="brush: powershell; title: ; notranslate" title="">Test-SQLLogin -SqlLogin 'domain\sqluser'
Test-SQLLogin -SqlLogin 'sqlsauser'
</pre>