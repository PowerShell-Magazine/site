---
title: '#PSTip Creating extended properties on database objects using SMO'
author: Hemanth C Damecharla
type: post
date: 2013-08-27T18:00:46+00:00
url: /2013/08/27/pstip-creating-extended-properties-on-database-objects-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

You might have seen several articles on the web discussing about the SQL database extended properties from a self-documentation point of view. I believe that we can use extended properties for much more than self-documentation.

Here is an example. Lets say you make a change to a DB configuration as part of a request. If you add the details and date of the change to the DB then, you exactly know what to revert when you have to do it 3-4 weeks down the line.

So, let us see how we can add extended properties using PowerShell.

```
Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
$extproperty = New-Object Microsoft.SqlServer.Management.Smo.ExtendedProperty

$extproperty.Parent = $server.Databases['sqlchow'] #needs smo object.
$extproperty.Name = 'Change made'
$extproperty.Value = 'Recovery Model set to BulkLogged'
$extproperty.Create()
```

In this tip we are adding extended properties to a DB, you can add extended properties to other objects as well. For example, if you need to add extended properties to a table, the Parent is going to be:

<pre class="brush: powershell; title: ; notranslate" title="">$server.Databases['sqlchow'].Tables['userdetails']
</pre>