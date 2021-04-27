---
title: '#PSTip Handling negative TimeSpan objects'
author: Shay Levy
type: post
date: 2013-02-18T19:01:51+00:00
url: /2013/02/18/pstip-handling-negative-timespan-objects/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Let&#8217;s say you created a new _TimeSpan_ object by using the _Start_ parameter.

```
PS> $ts = New-TimeSpan -Start 1/1/2014
PS> $ts

Days              : -324
Hours             : -9
Minutes           : -44
Seconds           : -12
Milliseconds      : -596
Ticks             : -280286525960331
TotalDays         : -324.405701342976
TotalHours        : -7785.73683223142
TotalMinutes      : -467144.209933885
TotalSeconds      : -28028652.5960331
TotalMilliseconds : -28028652596.0331
```

You ended up with an object whose value is negative because the start date is greater than the end date. PowerShell assigns the current date to the _End_ parameter if no value has been specified. So, what do you do if you need a positive value? If you send the object to the _Get-Member_ cmdlet you&#8217;ll notice a method called _Negate_.

```
PS> $ts | Get-Member -MemberType Method
   TypeName: System.TimeSpan
Name        MemberType Definition
----        ---------- ----------
Add         Method     timespan Add(timespan ts)
CompareTo   Method     int CompareTo(System.Object value), int CompareTo(timespan value), int IComparable.CompareTo(...
Duration    Method     timespan Duration()
Equals      Method     bool Equals(System.Object value), bool Equals(timespan obj), bool IEquatable[timespan].Equals...
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
Negate      Method     timespan Negate()
Subtract    Method     timespan Subtract(timespan ts)
ToString    Method     string ToString(), string ToString(string format), string ToString(string format, System.IFor...
```

All you need to do is invoke it.

```
PS> $ts.Negate()

Days              : 324
Hours             : 9
Minutes           : 44
Seconds           : 12
Milliseconds      : 596
Ticks             : 280286525960331
TotalDays         : 324.405701342976
TotalHours        : 7785.73683223142
TotalMinutes      : 467144.209933885
TotalSeconds      : 28028652.5960331
TotalMilliseconds : 28028652596.0331
```

Likewise, invoking the method on a positive object gives you back a negative instance.