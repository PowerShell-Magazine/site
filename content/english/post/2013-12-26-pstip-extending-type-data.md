---
title: '#PSTip Extending Type Data'
author: Shay Levy
type: post
date: 2013-12-26T19:00:41+00:00
url: /2013/12/26/pstip-extending-type-data/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

In the [previous article][1] Matt used lambda functions to simplify data manipulation . In this tip I want to show you a way of using functions as methods by extending the _System.Array_ class.

Beginning in Windows PowerShell 3.0 we can use the _Update-TypeData_ to update types with new type data. In previous versions of PowerShell this operation required writing a ps1xml file (lots of XML to deal with). Here&#8217;s an example that adds a _&#8216;map&#8217; script _method to _Array_ objects :

```
Update-TypeData -Force -MemberType ScriptMethod -MemberName map -TypeName System.Array -Value {
    param([object]$o)
    if($o -is [int]) { $this | foreach {$_ * $o} }
}

@(1..5).map(2)

2
4
6
8
10
```

One caveat of the new added method is its signature. When viewed using the Get-Member <span style="line-height: 1.5em;">cmdlet we can see that the method doesn&#8217;t accept any arguments though the script block used to extend the type is using a param block with one parameter (</span><em style="line-height: 1.5em;">$object</em><span style="line-height: 1.5em;">).</span>

```
PS> Get-Member -InputObject @(1..5) -MemberType ScriptMethod

   TypeName: System.Object[]
Name MemberType   Definition
---- ----------   ----------
map  ScriptMethod System.Object map();
```

In the same manner , we can use _Update-TypeData_ to &#8220;attach&#8221; new functionality to .NET types. Check the help of _Update-TypeData_ for more information and code examples.

[1]: /2013/12/23/simplifying-data-manipulation-in-powershell-with-lambda-functions/