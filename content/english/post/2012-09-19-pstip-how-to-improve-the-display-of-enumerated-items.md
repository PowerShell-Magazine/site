---
title: '#PSTip How to improve the display of enumerated items'
author: Shay Levy
type: post
date: 2012-09-19T18:00:07+00:00
url: /2012/09/19/pstip-how-to-improve-the-display-of-enumerated-items/
views:
  - 5486
post_views_count:
  - 953
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When working with objects in the console you may have seen a display that resembles the following:

```
PS> Get-Process s* | Select-Object Name,Threads -First 5
Name           Threads
----           -------
SearchIndexer  {5092, 2732, 1100, 2040...}
services       {656, 700, 764, 6100...}
shstat         {4440, 4672, 4196, 4232...}
SkyDrive       {2696, 4352, 4604, 448...}
smss           {268, 428}
```

Notice that the Threads column is showing 4 thread values and then it adds an ellipsis (&#8230;) to indicate that there are more items that are not shown.

PowerShell do this regardless of console window size.

PowerShell determines how many items to include in the the display using the $FormatEnumerationLimit preference variable.

As you already seen, it&#8217;s default value is 4, but we can change it and display more values.

```
PS> $FormatEnumerationLimit = 7
PS> Get-Process s* | Select-Object Name,Threads -First 5

Name           Threads
----           -------
SearchIndexer  {5092, 2732, 1100, 2040, 400, 3036, 3120...}
services       {656, 700, 764, 5168, 1612, 5104, 6560}
shstat         {4440, 4672, 4196, 4232, 3548, 2384, 1872...}
SkyDrive       {2696, 4352, 4604, 448, 1816, 2332, 1688...}
smss           {268, 428}
```

Want to reveal all values regardless of their count? Set $FormatEnumerationLimit to -1 (undocumented).

Note: The value does not affect the underlying objects, just the display.