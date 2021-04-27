---
title: '#PSTip Check if a SQL endpoint exists or not'
author: Ravikanth C
type: post
date: 2013-06-04T18:00:00+00:00
url: /2013/06/04/pstip-check-if-a-sql-endpoint-exists-or-not/
categories:
  - SQL
  - Tips and Tricks
tags:
  - SQL
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Before we make an attempt at creating a new [database mirroring endpoint][1], we need to verify if an endpoint with the given name and/or port already exists. We cannot create an endpoint with the same name or at the same port number. So, how do we validate this?

For checking if an endpoint with the same name exists or not, we can simply index into the [Endpoints property][2] of the SQL SMO [Server class][3].

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server -ArgumentList $env:computername
$smo.endpoints["MyEndPoint"]
</pre>

The above code returns the endpoint object, if it exists. We can use this as a condition to check the endpoint existence. For example,

<pre class="brush: powershell; title: ; notranslate" title="">if ($smo.endpoints["MyEndPoint"]) {
    #Do something
}
</pre>

But, how do we check if the port number we intend to use for the new endpoint is already in use or not? Well, let us use the Get-SQLEndpoint function we created in an [earlier tip][4].

<pre class="brush: powershell; title: ; notranslate" title="">if (!((Get-SQLEndpoint -ComputerName "MySQLServer" -InstanceName "MyInstance").ListenerPort -contains 7777)) {
    #Create endpoint
} else {
    Write-Error "An endpoint with port number 7777 already exists"
}
</pre>

[1]: http://msdn.microsoft.com/en-us/library/ms179511.aspx
[2]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.server.endpoints.aspx
[3]: http://msdn.microsoft.com/en-us/library/ms220151.aspx
[4]: /2013/06/03/pstip-list-all-endpoints-in-a-sql-deployment