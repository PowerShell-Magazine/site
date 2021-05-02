---
title: Finding Old Files
author: Shay Levy
type: post
date: 2011-09-11T14:11:52+00:00
url: /2011/09/11/finding-old-files/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When it comes to finding old files in PowerShell we have several options, all in which involve subtracting DateTime objects or using comparison operators. For example, if we want to remove all files older than 14 days, we can test if the LastWriteTime property of each FileInfo object is greater than a DateTime objects which has been subtracted 14 days:

```powershell
PS> $date = (Get-Date).AddDays(-14)
PS> Get-ChildItem -Path D:\Temp -Recurse |
    Where-Object {-not $_.PsIsContainer -and $_.LastWriteTime -lt $date } |
    Remove-Item -WhatIf
```


In the above example we initialize $date to a date in the past and test against it inside Where-Object.  When the older file is found, based on its LastWriteTime property, it is passed through to the Remove-Item cmdlet.

However, this doesn&#8217;t tell as the age of the file. We can produce a list of old files and create a new calculated property that will calculate the age:

```powershell
PS> $age = @{Name='Age(Days)';Expression={((Get-Date) - $_.LastWriteTime).Days}}
PS> Get-ChildItem | Select-Object -Property Name,FullName,$age
```


Now we get a 3-column output with an age of each file system object (including directories).

The expression we are using subtracts the LastWriteTime value of each file from the current date, which produces a TimeSpan object, and then we call its Days property to get the result.

But, did you know that you can also pipe file system objects to the New-TimeSpan cmdlet and it will give you the same result?

```powershell
PS> Get-ChildItem -Path $env:WINDIR\system.ini | New-TimeSpan

Days              : 137
Hours             : 4
Minutes           : 34
Seconds           : 54
Milliseconds      : 274
Ticks             : 118532942742544
TotalDays         : 137.190905952019
TotalHours        : 3292.58174284844
TotalMinutes      : 197554.904570907
TotalSeconds      : 11853294.2742544
TotalMilliseconds : 11853294274.2544
```

As you can see, system.ini is 137 days old. So, how does this work? Let&#8217;s take a close look of the Start parameter of New-TimeSpan:

```powershell
PS> (Get-Command New-TimeSpan).Parameters['Start']            

Name            : Start
ParameterType   : System.DateTime
ParameterSets   : {[Date, System.Management.Automation.ParameterSetMetadata]}
IsDynamic       : False
Aliases         : {LastWriteTime}
Attributes      : {System.Management.Automation.AliasAttribute, Date}
SwitchParameter : False
```

The Aliases member has the LastWriteTime alias defined, meaning that if an incoming object has a property of that name it will be automatically bind to the Start parameter. Pretty neat. So, based on the above we can also write a shorter expression:

```powershell
PS> $age = @{Name='Age(Days)';Expression={($_ | New-TimeSpan).Days}}
PS> Get-ChildItem | Select-Object -Property Name,FullName,$age
```
