---
title: '#PSTip Deleting extended properties on database objects using SMO'
author: Hemanth C Damecharla
type: post
date: 2013-08-30T18:00:26+00:00
url: /2013/08/30/pstip-deleting-extended-properties/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In earlier tips, we looked at how to [add][1], [read][2] and [update][3] the extended properties of a SQL database. In this tip we will see how we can delete them, in-case we need to do some cleanup.

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$server.Databases["sqlchow"].ExtendedProperties | Select Name, Value, State

Name          Value                           State
----          -----                           -----
Change made   Set recovery model to simple Existing
Change madeby Crack-Monkey                 Existing

#drop the extended property
$server.Databases["sqlchow"].ExtendedProperties["Change made"].Drop()
$server.Databases["sqlchow"].ExtendedProperties | Select Name, Value, State

Name          Value           State
----          -----           -----
Change madeby Crack-Monkey Existing
```

[1]: /2013/08/23/pstip-creating-extended-properties-on-database-objects-using-smo/
[2]: /2013/08/23/pstip-reading-extended-properties-on-database-objects-using-smo/
[3]: /2013/08/23/pstip-updating-extended-properties-on-database-objects-using-smo/