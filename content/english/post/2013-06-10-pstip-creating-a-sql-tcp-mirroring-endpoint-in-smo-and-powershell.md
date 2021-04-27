---
title: '#PSTip Creating a SQL TCP mirroring endpoint with SMO and PowerShell'
author: Ravikanth C
type: post
date: 2013-06-10T18:00:00+00:00
url: /2013/06/10/pstip-creating-a-sql-tcp-mirroring-endpoint-in-smo-and-powershell/
categories:
  - SQL
  - Tips and Tricks
tags:
  - SQL
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In today&#8217;s tip, we will see how we can use PowerShell and SMO to create a SQL TCP mirroring endpoint. We can use the [Create()][1] method of [Microsoft.SqlServer.Management.Smo.Endpoint][2] class to create an endpoint. In the following function, I used the code we published in [earlier tips][3] to verify whether an endpoint exists or not. So, to be able to use this function, you need the [Get-SQLEndpoint][4] function from an earlier tip.

<pre class="brush: powershell; title: ; notranslate" title="">


    Function New-SQLMirroringTCPEndpoint {
        [CmdletBinding()]
        param (
            [string]$computername=$env:COMPUTERNAME,
            [string]$instancename,
            [string]$endpointname,
            [int]$endpointport
        )
        Begin {
            Write-Verbose "Loading SQL SMO"
            Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
            Add-Type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
        }
    
        Process {
                try {
                    $connection = New-Object Microsoft.SqlServer.Management.Common.ServerConnection -ArgumentList $ComputerName
                    $connection.applicationName = "PowerShell SQL SMO"
    
                    if ($instancename) {
                        Write-Verbose "Connecting to SQL named instance"
                        $connection.ServerInstance = "${env:computername}\${instancename}"
                    } else {
                        Write-Verbose "Connecting to default SQL instance"
                    }
    
                    $connection.StatementTimeout = 0
                    $connection.Connect()
                    $smo = New-Object Microsoft.SqlServer.Management.Smo.Server -ArgumentList $connection
                }
                catch {
                    Write-Error $_
                }
    
                try {
                    if (!($smo.Endpoints[$endpointname])) {
                        if (!((Get-SQLEndPoint -computername $computername -instancename $instancename).ListenerPort -contains $endpointport)) {
                            Write-Verbose "Creating a mirroring endpoint named ${endpointname} at port ${endpointport}"
                            $SQLEndPoint = New-Object Microsoft.SqlServer.Management.Smo.Endpoint -ArgumentList $smo, $endpointname
                            $SQLEndPoint.EndpointType = [Microsoft.SqlServer.Management.Smo.EndpointType]::DatabaseMirroring
                            $SQLEndPoint.ProtocolType = [Microsoft.SqlServer.Management.Smo.ProtocolType]::TCP
                            $SQLEndPoint.Protocol.Tcp.ListenerPort = $endpointport
                            $SQLEndPoint.Payload.DatabaseMirroring.ServerMirroringRole = [Microsoft.SqlServer.Management.Smo.ServerMirroringRole]::All
                            $SQLEndPoint.Create()
                            $SQLEndPoint.Start()
                            $smo.Endpoints[$endpointname]
                        } else {
                            Write-Error "An endpoint with specified port number ${endpointport} already exists"
                        }
                    } else {
                        Write-Error "An endpoint with name ${endpointname} already exists"
                    }
                }
    
                catch {
                    Write-Error $_
                }
        }
    }
The way we use this function is simple.

```
#To create an endpoint on the local computer with default SQL instance
New-SQLMirroringTCPEndpoint -endpointname TestEndPoint -endpointport 8888

#To create an endpoint on a remote computer with  a named SQL instance
New-SQLMirroringTCPEndpoint -computername server01 -instancename mySQLInstance -endpointname testendpoint -endpointport 9999
```

[1]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.endpoint.create.aspx
[2]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.endpoint.aspx
[3]: /tag/smo/
[4]: /2013/06/03/pstip-list-all-endpoints-in-a-sql-deployment/