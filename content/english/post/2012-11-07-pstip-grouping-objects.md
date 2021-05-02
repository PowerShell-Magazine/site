---
title: '#PSTip Grouping objects'
author: Shay Levy
type: post
date: 2012-11-07T19:00:22+00:00
url: /2012/11/07/pstip-grouping-objects/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The Group-Object cmdlet lets you group objects based on the value of a specified property. For example:

```
PS> Get-Service | Group-Object -Property Status
Count Name     Group
----- ----     -----
   93 Stopped  {AeLookupSvc, AllUserInstallAgent, AppIDSvc, AppMgmt...}
   81 Running  {ALG, Appinfo, Apple Mobile Device, AudioEndpointBuilder...}
```

This command gets the services and groups them by status. In addition to the Name and Count properties, you get back the objects of each group. If all you need is a Name/Count pair then you can tell Group-Object to omit the members of each group. The command will return the result much faster.

```
PS> Get-ChildItem $env:WINDIR | Group-Object -Property Extension -NoElement
Count Name
----- ----
   67
    2 .Tmp
    1 .NET
    9 .exe
    1 .dat
    6 .log
    2 .xml
    1 .bin
    2 .ini
    1 .dll
    1 .prx
```

Sometimes you&#8217;d need to group objects based on a part of a property value. Luckily, Group-Object enables us to do so using an expression. The following command will help you to find out how many files in the current directory were changed per month:

```
PS> Get-ChildItem | Group-Object { $_.LastWriteTime.Month } -NoElement | Sort-Object Name
Count Name
----- ----
   37 1
   45 10
   33 11
   36 12
   22 2
   27 3
   19 4
   29 5
   37 6
   34 7
  190 8
   46 9
```