---
title: '#PSTip Updating extended properties on database objects using SMO'
author: Hemanth C Damecharla
type: post
date: 2013-08-29T18:00:53+00:00
url: /2013/08/29/pstip-updating-extended-properties-on-database-objects-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

In an [earlier tip][1], we saw how we can read extended properties to a database. In this tip we see how we can alter them.

We can use extended properties to [keep a record of the configuration changes made to the DB][2]. Assuming we already have a set of extended properties called &#8216;Change made&#8217; and &#8216;Change made by&#8217;, let us see how we can update these properties.

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$server.Databases["sqlchow"].ExtendedProperties | Select Name, Value, State

Name          Value                               State
----          -----                               -----
Change made   Recovery Model set to BulkLogged Existing
Change madeby CandidConvos                     Existing

$proptochange  = $server.Databases["sqlchow"].ExtendedProperties["Change made by"]
$proptochange.Value = "Crack-Monkey"
$proptochange.Alter()
$server.Databases["sqlchow"].ExtendedProperties | Select Name, Value, State

Name          Value                               State
----          -----                               -----
Change made   Recovery Model set to BulkLogged Existing
Change madeby Crack-Monkey                     Existing
```

[1]: /2013/08/23/pstip-reading-extended-properties-on-database-objects-using-smo/
[2]: /2013/08/23/pstip-creating-extended-properties-on-database-objects-using-smo/