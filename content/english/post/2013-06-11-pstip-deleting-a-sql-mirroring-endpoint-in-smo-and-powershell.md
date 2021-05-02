---
title: '#PSTip Deleting a SQL mirroring endpoint with SMO and PowerShell'
author: Ravikanth C
type: post
date: 2013-06-11T18:00:00+00:00
url: /2013/06/11/pstip-deleting-a-sql-mirroring-endpoint-in-smo-and-powershell/
categories:
  - SQL
  - Tips and Tricks
tags:
  - SQL
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In earlier tips, we looked at how we can get [all SQL endpoints][1], check if a [SQL endpoint exists or not][2], and [creating SQL mirroring endpoints][3]. In this tip, we will see how we can delete the existing SQL endpoints.


    Function Remove-SQLEndPoint {
        [CmdletBinding()]
        param (
            [string]$computername=$env:COMPUTERNAME,
            [string]$instancename,
            [string]$endpointname
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
                    if ($smo.Endpoints[$endpointname]) {
                        Write-Verbose "Dropping the endpoint ${endpointname}"
                        $smo.Endpoints[$endpointname].Drop()
                    } else {
                        Write-Error "No end point exists with the name ${endpointname}"
                    }
                }
    
                catch {
                    Write-Error $_
                }
        }
    }
For the purpose of removing the endpoints, we use the [Drop()][4] method of the [Endpoint class][5]. For error handling purpose, we check if the given endpoint name exists within all existing endpoints on the SQL server.

[1]: /2013/06/03/pstip-list-all-endpoints-in-a-sql-deployment/
[2]: /2013/06/04/pstip-check-if-a-sql-endpoint-exists-or-not/
[3]: /2013/06/10/pstip-creating-a-sql-tcp-mirroring-endpoint-in-smo-and-powershell
[4]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.endpoint.drop.aspx
[5]: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.endpoint.aspx