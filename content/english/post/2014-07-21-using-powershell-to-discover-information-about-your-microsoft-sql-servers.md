---
title: Using PowerShell to discover information about your Microsoft SQL Servers
author: Mike Robbins
type: post
date: 2014-07-21T16:00:09+00:00
url: /2014/07/21/using-powershell-to-discover-information-about-your-microsoft-sql-servers/
categories:
  - How To
  - SQL
tags:
  - How To
  - SQL

---
I’m an infrastructure guy who supports many different products at multiple datacenters in an enterprise environment. One of those products is Microsoft SQL Server which I’ve been supporting since version 6.5 back in the 1990s. In this article, I’ll be discussing how PowerShell can be used to retrieve just about any information that you would want to know about modern versions of SQL Server that you currently have running in your environment. This article isn’t meant to be a deep dive, it’s meant to get you started thinking about how you could write your own PowerShell code to retrieve the specific information that you’re looking for from your SQL Servers.

In this scenario, you have two servers running the core installation (no-GUI) version of Windows Server 2012 R2. One has SQL Server 2008 R2 installed and the other one has SQL Server 2014 installed, each one has multiple instances of SQL Server installed. All of the examples shown are being performed on a Windows 8.1 Enterprise edition workstation with the SQL Server 2014 management tools installed and the RSAT (Remote Server Administration Tools) installed.

Note: SQL Server 2008 R2 and prior versions of SQL Server that supported PowerShell use a snap-in and beginning with SQL Server 2012 a module is used instead.

**Working with SQL Server as if it were a file system**

One way of working with SQL Server in PowerShell is through the SQLServer PSDrive that’s created when the SQL PowerShell snap-in or module is imported. This allows you to work with SQL Server as if it were a file system.

Since the SQL Server 2014 client tools are installed on our workstation, there is a PowerShell module named _SQLPS_ installed and you’ll need to start out by importing that module:

```
Import-Module -Name SQLPS -DisableNameChecking
```

![](/images/sql12.png)

Notice that the -DisableNameChecking parameter was specified in the previous example. There are a couple of cmdlets in the SQLPS module that use unapproved verbs (Encode-SqlName and Decode-SqlName) that will cause warnings to be generated if this optional parameter isn’t specified. The warnings wouldn’t hurt anything, but I prefer not to see them. You can also see in the previous example that when the SQLPS module is imported, it automatically changes your current location to the SQLSERVER PSDrive.

Determining the instances for the server named SQL01 is simple as shown in the following example:

```
Get-ChildItem -Path 'SQLSERVER:\SQL\SQL01'
```

![](/images/sql121.png)

One of the things that makes PowerShell so powerful is that once you figure out how to perform a task for one item (one computer in this scenario), it’s easy to perform that same task for multiple items.

The ForEach-Object cmdlet can be used to return the results for multiple SQL Servers:

```
'SQL01', 'SQL02' |
ForEach-Object {Get-ChildItem -Path "SQLSERVER:\SQL\$_"}
```

![](/images/sql122.png)

By default, only the InstanceName property is returned, so you have no idea which instances belong to which servers.

Just like any other cmdlet that produces output, you can pipe the previous command to Get-Member to see all of the available properties or Format-List –Properties * to see all of the available properties and their values. Be prepared to be overwhelmed though because there’s a lot more information about SQL Server that can be obtained. For the sake of simplicity, I chose to focus on the Instance Name so here are a few helpful properties:

'SQL01', 'SQL02' |
ForEach-Object {Get-ChildItem -Path "SQLSERVER:\SQL\$_"} |
Select-Object -Property ComputerNamePhysicalNetBIOS, Name, DisplayName, InstanceName

![](/images/sql123.png)

You may have too many SQL Servers to manually type in the name for, but that’s not a problem if they’re stored in Active Directory where you can query the names from such as in their own Active Directory OU (Organizational Unit):

```
Get-ADComputer -Filter * -SearchBase 'OU=SQL Servers,OU=Computers,OU=Test,DC=mikefrobbins,DC=com' |
Select-Object -ExpandProperty Name |
ForEach-Object {Get-ChildItem -Path "SQLSERVER:\SQL\$_"} |
Select-Object -Property ComputerNamePhysicalNetBIOS, DisplayName
```

![](/images/sql124.png)

You could also return information such as when the latest backups were taken and what the recovery model is for every database on every instance of every SQL Server all with a PowerShell one-liner:

```
Get-ADComputer -Filter * -SearchBase 'OU=SQL Servers,OU=Computers,OU=Test,DC=mikefrobbins,DC=com' |
Select-Object -ExpandProperty Name |
ForEach-Object {Get-ChildItem -Path "SQLSERVER:\SQL\$_"} |
ForEach-Object {Get-ChildItem -Path "SQLSERVER:\SQL\$($_.ComputerNamePhysicalNetBIOS)\$($_.DisplayName)\Databases" -Force} |

Select-Object -Property @{label='ServerName';expression={($_.Parent -replace '^\[|\]$|\\.*$').ToUpper()}}, Name, LastBackupDate, LastDifferentialBackupDate, LastLogBackupDate, RecoveryModel |

Format-Table -AutoSize
```

![](/images/sql125.png)

Although the command is on more than one physical line, it’s still considered to be a PowerShell one-liner because it’s one continuous pipeline.

All of the previous examples run against one server at a time and if a server isn’t responding, you would have to wait for it to time out before the next one would begin which could be a slow process depending on how many SQL Servers are in your environment. Beginning with Windows Server 2012, PowerShell remoting is enabled by default so we could simply wrap the previous examples inside the <em>Invoke-Command</em> cmdlet which would run the commands against up to 32 servers in parallel by default. The number of servers that <em>Invoke-Command</em> runs against in parallel at a time is also configurable via the ThrottleLimit parameter. The only modification that would need to be made is the commands inside of the <em>Invoke-Command</em> script block would need to target the local computer since it would effectively be running locally on the remote SQL Servers,<a href="https://red9.com/sql-server-consulting/"> Microsoft SQL Server Consultancy</a> can help with easily with this process, and the results would be returned as deserialized objects.

### Retrieving SQL Instance Names from the Registry

The name of each instance could also be obtained from the registry of the SQL Servers:

```
$SQLInstances = Invoke-Command -ComputerName sql01, sql02 {
Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server'
}
foreach ($SQLInstance in $SQLInstances) {
    foreach ($s in $SQLInstance.InstalledInstances) {
       [PSCustomObject]@{
           PSComputerName = $SQLInstance.PSComputerName
           InstanceName = $s
       }
   }
}
```

![](/images/sql126.png)

### Retrieving SQL Instance Names with WMI (Windows Management Instrumentation)

You could also obtain the SQL instance information with WMI. Be aware that different versions of SQL Server use different WMI namespaces.

```
Get-CimInstance -ComputerName sql01 -Namespace 'root/Microsoft/SqlServer/ComputerManagement12' -ClassName ServerSettings
```

![](/images/sql127.png)

### Retrieving SQL Instance Names from Services

If you’ve worked with PowerShell for more than 10 minutes, then you’ve probably been introduced to the <em>Get-Service</em> cmdlet. Believe it or not, the <em>Get-Service</em> cmdlet is one of the simplest ways to get a list of the instances on your SQL Servers since every instance that’s installed creates its own service:

```
Invoke-Command -ComputerName sql01, sql02 {
    Get-Service -Name MSSQL* |
    Where-Object Status -eq 'Running'
} | Select-Object -Property PSComputerName, @{label='InstanceName';expression={$_.Name -replace '^.*\$'}}
```

![](/images/sql128.png)

### Running existing T-SQL from PowerShell

If you have existing T-SQL statements, you can simply wrap them inside of the Invoke-Sqlcmd cmdlet:

Invoke-Sqlcmd -ServerInstance sql01 -Database master -Query 'SELECT name FROM sys.databases'

![](/images/sql129.png)

### Running Stored Procedures from PowerShell

You can also run stored procedures from PowerShell using the Invoke-SQLCmd cmdlet:

```
Invoke-Sqlcmd -ServerInstance sql01 -Database master -Query 'EXEC sp_databases'
```

![](/images/sql130.png)

### Working with SQL Server through the use of SMO (SQL Server Management Objects)

SMO in my opinion is a little more complicated, but it’s also one of the more popular methods for accessing information about your SQL Servers with PowerShell.

You may have seen other articles where DLLs had to be imported in order to use SMO and that’s still the case if you’re running a version of the SQL Server management tools that uses a PowerShell snap-in, but the good news is that beginning with SQL Server 2012 where a PowerShell module is used, the necessary DLLs are automatically imported when the module is imported.

In this example, I’ll use SMO to return a list of database names for the default instance of SQL Server on sql01:

```
$SQL = New-Object('Microsoft.SqlServer.Management.Smo.Server') -ArgumentList 'SQL01'
$SQL.Databases.Name
```

![](/images/sql131.png)

You could create reusable PowerShell functions that leverage SMO to accomplish your common tasks such as this one that retrieves the backup information for one or more SQL Servers and instances:

```
#Requires -Version 3.0
function Get-MrDbBackupInfo {
<#
.SYNOPSIS
Returns database backup information for a Microsoft SQL Server database.

.DESCRIPTION
Get-DbBackupInfo is a function that returns database backup information for
one or more Microsoft SQL Server databases.

.PARAMETER ComputerName
The computer that is running Microsoft SQL Server that you’re targeting to
query database backup information for.

.PARAMETER InstanceName
The instance name of SQL Server to return database backup information for.
The default is the default SQL Server instance.

.PARAMETER DatabaseName
The database(s) to return backup information for. The default is all databases.

.EXAMPLE
Get-DbBackupInfo -ComputerName sql01

.EXAMPLE
Get-DbBackupInfo -ComputerName sql01 -DatabaseName master, msdb, model

.EXAMPLE
Get-DbBackupInfo -ComputerName sql01 -InstanceName MrSQL -DatabaseName master,msdb, model

.EXAMPLE
'master', 'msdb', 'model' | Get-DbBackupInfo -ComputerName sql01

.INPUTS
String

.OUTPUTS
PSCustomObject
#>

   [CmdletBinding()]
   param (
      [Parameter(Mandatory,
      ValueFromPipelineByPropertyName)]
      [Alias('ServerName','PSComputerName')]
      [string[]]$ComputerName,

      [Parameter(ValueFromPipelineByPropertyName)]
      [ValidateNotNullOrEmpty()]
      [string[]]$InstanceName = 'Default',
    
      [Parameter(ValueFromPipelineByPropertyName)]
      [ValidateNotNullOrEmpty()]
      [string[]]$DatabaseName = '*'

   )

   BEGIN {
      $problem = $false
      Write-Verbose -Message "Attempting to load SQL Module if it's not already loaded"
      if (-not (Get-Module -Name SQLPS)) {
          try {
              Import-Module -Name SQLPS -DisableNameChecking -ErrorAction Stop
          }
          catch {
              $problem = $true
              Write-Warning -Message "An error has occurred.&amp;nbsp; Error details: $_.Exception.Message"
          }
      }
   }

   PROCESS {
       foreach ($Computer in $ComputerName) {
            foreach ($Instance in $InstanceName) {
                Write-Verbose -Message 'Checking for default or named SQL instance'
                If (-not ($problem)) {
                    If (($Instance -eq 'Default') -or ($Instance -eq 'MSSQLSERVER')) {
                       $SQLInstance = $Computer
                    }
                    else {
                       $SQLInstance = "$Computer\$Instance"
                    }
                    $SQL = New-Object('Microsoft.SqlServer.Management.Smo.Server') -ArgumentList $SQLInstance
                }

                if (-not $problem) {
                     foreach ($db in $DatabaseName) {
                         Write-Verbose -Message "Verifying a database named: $db exists on SQL Instance $SQLInstance."
                         try {
                             if ($db -match '\*') {
                                  $databases = $SQL.Databases | Where-Object Name -like "$db"
                             }
                             else {
                                  $databases = $SQL.Databases | Where-Object Name -eq "$db"
                             }
                         }
                         catch {
                             $problem = $true
                             Write-Warning -Message "An error has occurred.&amp;nbsp; Error details: $_.Exception.Message"
                         }
                         if (-not $problem) {
                             foreach ($database in $databases) {
                                  Write-Verbose -Message "Retrieving information for database: $database."
                                  [PSCustomObject]@{
                                      ComputerName = $SQL.Information.ComputerNamePhysicalNetBIOS
                                      InstanceName = $Instance
                                      DatabaseName = $database.Name
                                      LastBackupDate = $database.LastBackupDate
                                      LastDifferentialBackupDate = $database.LastDifferentialBackupDate
                                      LastLogBackupDate = $database.LastLogBackupDate
                                      RecoveryModel = $database.RecoveryModel
                             }
                          }
                     }
                 }
             }
         }
      }
   }
}

Get-MrDbBackupInfo -ComputerName sql01 | Format-Table
```

![](/images/sql132.png)

Each time you find a task that you commonly need to perform for your SQL Servers, write a PowerShell function for it, combine those functions into a script module and you’ll have your own custom SQL Server PowerShell Toolkit. Last but not least, share your toolkit with the PowerShell and SQL Server communities.

