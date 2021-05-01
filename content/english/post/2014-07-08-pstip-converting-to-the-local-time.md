---
title: '#PSTip Converting to the local time'
author: Ravikanth C
type: post
date: 2014-07-08T18:00:13+00:00
url: /2014/07/08/pstip-converting-to-the-local-time/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Several RSS feeds that I refer regularly have a publication date set to another time zone. For my own tracking purpose, I needed a way to convert it to the local time zone and store the date. There are several ways to do this but the following are my favorite ways of achieving this and in that order.

### Using DateTime

The [_Parse_][1] method of DateTime .NET class can be used in PowerShell to convert between different time zones.

```
PS> [DateTime]::Parse('Mon, 07 Jul 2014 18:00:22 +0000')
Monday, July 07, 2014 11:30:22 PM

PS> [DateTime]::Parse('Thu, 21 Nov 2013 22:40:12 GMT')
Friday, November 22, 2013 4:10:12 AM

PS> [DateTime]"Mon, 07 Jul 2014 18:00:22 +0000"
Monday, July 07, 2014 11:30:22 PM

PS> [DateTime]"Thu, 21 Nov 2013 22:40:12 GMT"
Friday, November 22, 2013 4:10:12 AM
```

### Using TimeZone.CurrentTimeZone

We can also use the [ToLocalTime][2] method of the TimeZone .NET class:

```
PS> [System.TimeZone]::CurrentTimeZone.ToLocalTime('Mon, 07 Jul 2014 18:00:22 +0000')
Monday, July 07, 2014 11:30:22 PM

PS> [System.TimeZone]::CurrentTimeZone.ToLocalTime('Thu, 21 Nov 2013 22:40:12 GMT')
Friday, November 22, 2013 4:10:12 AM
```

What is your favorite way of doing this?

[1]: http://msdn.microsoft.com/en-us/library/system.datetime.parse(v=vs.110).aspx
[2]: http://msdn.microsoft.com/en-us/library/system.timezone.tolocaltime(v=vs.110).aspx