---
title: '#PSTip Overriding the ToString() Method, part II'
author: Shay Levy
type: post
date: 2013-12-19T19:00:41+00:00
url: /2013/12/19/pstip-overriding-the-tostring-method-part-ii/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In this [neat tip][1] Jakub showed a way to control the output of the _ToString_ method by exposing a ToString function. What if you can&#8217;t intervene with code or don&#8217;t have access to it? Consider the following example:

```
PS> $gps = Get-Process s* | Select-Object -First 3
PS> "The first 3 service names are: $gps"

The first 3 service names are: System.Diagnostics.Process (SearchIndexer) System.Diagnostics.Process (services) System.Diagnostics.Process (SettingSyncHost)
```

You can see that when the variable that holds the processes is converted to a string, the _ToString_ method of each process returns the .NET type name followed by the process name in parenthesis. Obviously this is not the way we want to present the data. Let&#8217;s see how we can override the _ToString_ method of _System.Diagnostics.Process_ (or any other object):

```
PS> $gps = Get-Process s* | Select-Object -First 3 | Add-Member -MemberType ScriptMethod -Name ToString -Value {$this.Name} -PassThru -Force
PS> "The first 3 service names are: $gps"

The first 3 service names are: SearchFilterHost SearchIndexer SearchProtocolHost
```

Excellent, now we get just the names. To introduce your own _ToString_ method you use the _Add-Member_ cmdlet to add a new _ScriptMethod_ called _ToString_.

In the _Value_ scriptblock, _$this_ represents the current object flowing through the cmdlet and we use it to return the _Name_ of the object (process object in our case).

Because each object already has a _ToString_ method we add the _Force_ switch to override it, and we also add the _PassThru_ switch to write the extended object back to the pipeline.

[1]: /2013/12/18/pstip-overriding-the-tostring-method