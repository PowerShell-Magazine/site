---
title: '#PSTip Enumerate all SQL Server instances in a network'
author: Ravikanth C
type: post
date: 2013-04-24T18:00:08+00:00
url: /2013/04/24/pstip-enumerate-all-sql-server-instances-in-a-network/
categories:
  - SQL
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 2.0 or above.

Ever thought about how you can list of all SQL Server instances on the local network? SQL Management Objects (SMO) provides a way to do that. We can use the _[EnumAvailableSqlServers()][1]_ Method in the _SmoApplication_ class to achieve this.

Let us see how:

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91";
[Microsoft.SqlServer.Management.Smo.SmoApplication]::EnumAvailableSqlServers()
</pre>

There are a couple of other variants of this method.

We can use _[EnumAvailableSqlServers($true)][1]_ to list only the local SQL Server instances. Using _$false_ as the method argument has the same effect as the above method of listing all instances on the network.

<pre class="brush: powershell; title: ; notranslate" title="">[Microsoft.SqlServer.Management.Smo.SmoApplication]::EnumAvailableSqlServers($true)
</pre>

And, [_EnumAvailableSqlServers(&#8220;MyDBServer&#8221;)_][2] will return SQL instances on the remote server named _MyDBServer_.

<pre class="brush: powershell; title: ; notranslate" title="">[Microsoft.SqlServer.Management.Smo.SmoApplication]::EnumAvailableSqlServers("MyDBServer")
</pre>

[1]: http://msdn.microsoft.com/en-us/library/ms210334.aspx
[2]: http://msdn.microsoft.com/en-us/library/ms210425.aspx