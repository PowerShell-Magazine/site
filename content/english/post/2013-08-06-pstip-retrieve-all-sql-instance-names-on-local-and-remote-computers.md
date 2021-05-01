---
title: '#PSTip Retrieve all SQL instance names on local and remote computers'
author: Ravikanth C
type: post
date: 2013-08-06T18:00:21+00:00
url: /2013/08/06/pstip-retrieve-all-sql-instance-names-on-local-and-remote-computers/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
I was looking at way to retrieve the SQL instance names on a remote computer without using SQL SMO. This is essential for me as I don&#8217;t always expect to have the SQL SMO DLLs on the computer where I run my scripts.

We can certainly use the Windows Registry to find this information. If the remote system has a SQL service running, the SQL instance information is stored under _HKLM\SOFTWARE\Microsoft\Microsoft SQL Server\Instance Names\SQL_ key. We can use the remote Registry service to query for this information from remote systems. So, having remote Registry service enabled on the target systems is essential for the following script to work.


    Function Get-SQLInstance {
        param (
            [string]$ComputerName = $env:COMPUTERNAME,
            [string]$InstanceName
        )
    
        try {
            $reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $ComputerName)
            $regKey= $reg.OpenSubKey("SOFTWARE\\Microsoft\\Microsoft SQL Server\\Instance Names\\SQL" )
            $instances = $regkey.GetValueNames()
    
            if ($InstanceName) {
                if ($instances -contains $InstanceName) {
                    return $true
                } else {
                    return $false
                }
            } else {
                $instances
            }
        }
        catch {
            Write-Error $_.Exception.Message
            return $false
        }
    }
You can use the above function as follows:

```
PS> Get-SQLInstance
MSSQLSERVER

PS C:\> Get-SQLInstance -ComputerName SQL1
MSSQLSERVER
MIRRORInstance

PS C:\> Get-SQLInstance -ComputerName SQL1 -InstanceName SQLServer
False
```