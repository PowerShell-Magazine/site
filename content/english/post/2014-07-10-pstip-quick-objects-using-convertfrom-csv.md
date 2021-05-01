---
title: '#PSTip Quick objects using ConvertFrom-Csv'
author: Bartek Bielawski
type: post
date: 2014-07-10T18:00:18+00:00
url: /2014/07/10/pstip-quick-objects-using-convertfrom-csv/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or later.

PowerShell is designed to operate on objects. But, what can we do if all we have is text e.g. from a ticket body or email?

My main tool at work is PowerShell ISE and it makes converting text data into custom object relatively easy. I just put my text data in here-string in the Script Pane and let ConvertFrom-Csv do the rest. There are two possible scenarios.

In the first one I have pieces of information from the different sources and I can format it as comma-separated values and convert it easily into custom objects:

<pre class="brush: powershell; title: ; notranslate" title="">@'
Name,Department,Title
John Doe,Finance,Accountant
Marry Jonson,IT,Consultant
Andrew Smith,HR,Recruiter
'@ | ConvertFrom-Csv
</pre>

In the second, and more frequent, scenario I have data that looks almost like result from Format-Table. There is no obvious delimiter: few spaces here, one space there. That forces me to clean up the data first, before I can convert it to objects:

<pre class="brush: powershell; title: ; notranslate" title="">@'
Name        Status            IP                  
WINSRV0001  Production        192.168.100.1       
WINSRV0002  Decommissioned    192.168.101.2       
LINSRV0001  Racked            192.168.102.2       
'@ -replace ' +', ',' | ConvertFrom-Csv
</pre>

The only limitation is that all properties will be strings, even if they look like numbers:

```
$data = @'
VM,DiskSizeGB
WINVM0001,40
WINVM0003,120
LINVM0004,15
LINVM0005,1100
'@ | ConvertFrom-Csv

$data | Get-Member -MemberType Properties

	TypeName: System.Management.Automation.PSCustomObject

Name       MemberType   Definition                 
----       ----------   ----------
DiskSizeGB NoteProperty System.String DiskSizeGB=40
VM         NoteProperty System.String VM=WINVM0001 
```

This may result in unexpected results when sorting/ filtering:

```
$data | Where-Object { 
    $_.DiskSizeGB -gt 100
} | Sort-Object DiskSizeGB 

VM                                      DiskSizeGB                             
--                                      ----------
LINVM0005                               1100                                   
WINVM0003                               120                                    
LINVM0004                               15                                     
WINVM0001                               40                                     
```

To fix **filtering** we have to remember that when comparing PowerShell will always convert value on the right to the type of value on the left. For **sorting** we have to use a script block and convert property to a number rather than use property as a string type:

```
$data | Where-Object { 
    100 -lt $_.DiskSizeGB
} | Sort-Object { [int]$_.DiskSizeGB } 

VM                                      DiskSizeGB                             
--                                      ----------
WINVM0003                               120                                    
LINVM0005                               1100                                   
```

