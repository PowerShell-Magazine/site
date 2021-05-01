---
title: '#PSTip Reading extended properties on database objects using SMO'
author: Hemanth C Damecharla
type: post
date: 2013-08-28T18:00:23+00:00
url: /2013/08/28/pstip-reading-extended-properties-on-database-objects-using-smo/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

Extended properties are an useful but, under utilized feature in SQL server. In an [earlier tip][1], we saw that you can add extended properties to databases and other objects. You can also add them to columns in your tables, and many other objects in your databases.

Now for the fun part. Let us say we have a database &#8216;sqlchow&#8217; in your instance. We can retrieve the extended properties, if they have been set, using the following code:


	Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
	$server = New-Object Microsoft.SqlServer.Management.Smo.Server $env:COMPUTERNAME
	$server.Databases["sqlchow"].ExtendedProperties | Select Name, Value, State
	Name                   Value                                   State
	----                   -----                                   -----
	CreatedBy              sqlchow                              Existing
	CreatedOn              2013-08-17 13:00:06                  Existing
	Generate Documentation 1                                    Existing
	Purpose                Demo creation of extended properties Existing
In the next tip let us see how we can alter extended properties on an object.

[1]: /2013/08/23/pstip-creating-extended-properties-on-database-objects-using-smo/