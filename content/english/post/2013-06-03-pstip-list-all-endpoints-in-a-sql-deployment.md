---
title: '#PSTip List all endpoints in a SQL deployment'
author: Ravikanth C
type: post
date: 2013-06-03T18:22:15+00:00
url: /2013/06/03/pstip-list-all-endpoints-in-a-sql-deployment/
categories:
  - SQL
  - Tips and Tricks
tags:
  - SQL
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

When working with SQL Server database mirroring, it is desired to understand how to create and troubleshoot database endpoints. As with many other things, we can use SQL SMO to work with endpoints. In today&#8217;s tip, I will show you how to retrieve all database endpoints including the mirroring endpoints.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$smo = New-Object Microsoft.SqlServer.Management.Smo.Server -ArgumentList $env:computername
$smo.endpoints
</pre>

The above code retrieves all endpoints on the local SQL server&#8217;s default instance. Now, let us extend the above code to retrieve database endpoints from any SQL Server in the network and from any instance.


    Function Get-SQLEndpoint {
        [CmdletBinding()]
        param (
            [string]$ComputerName=$env:COMPUTERNAME,
            [string]$InstanceName
        )
    	Begin {
            Write-Verbose "Loading SQL SMO"
            Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
            Add-Type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=11.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
        }
    
    	Process {
            try {
                $connection = New-Object Microsoft.SqlServer.Management.Common.ServerConnection -ArgumentList $ComputerName
                $connection.applicationName = "PowerShell SQL SMO"
    
                if ($InstanceName) {
                    Write-Verbose "Connecting to SQL named instance"
                    $connection.ServerInstance = "${ComputerName}\${InstanceName}"
                } else {
                    Write-Verbose "Connecting to default SQL instance"
                }
    
                $connection.StatementTimeout = 0
                $connection.Connect()
                $smo = New-Object Microsoft.SqlServer.Management.Smo.Server -ArgumentList $connection
                $smo.Endpoints | Select Name, EndPointType, ProtocolType, EndpointState, @{Name="ListenerPort";Expression={if ($_.ProtocolType -eq "TCP") {$_.Protocol.TCP.ListenerPort} else {$_.Protocol.HTTP.ListenerPort}}}
            }
            catch {
                Write-Error $_
            }
    	}
    }
You can use this function as shown below.

<pre class="brush: powershell; title: ; notranslate" title="">Get-SQLEndpoint -ComputerName "SQL-SERVER-01" -InstanceName "MyInstance"
Get-SQLEndpoint -ComputerName "SQL-SERVER-01" -InstanceName "MyInstance" | Where-Object { $_.EndPointType -eq "DatabaseMirroring" }
</pre>