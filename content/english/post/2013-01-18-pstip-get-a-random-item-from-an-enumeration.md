---
title: '#PSTip Get a random item from an enumeration'
author: Ravikanth C
type: post
date: 2013-01-18T19:01:56+00:00
url: /2013/01/18/pstip-get-a-random-item-from-an-enumeration/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier tip, I showed you how to get the [item count in an enumeration][1]. In this tip, we shall see how to retrieve a random item from an enumeration.

```
Add-Type -AssemblyName System.Drawing
$count = [Enum]::GetValues([System.Drawing.KnownColor]).Count
[System.Drawing.KnownColor](Get-Random -Minimum 1 -Maximum $count)
```


Simple!

[1]: /2013/01/17/pstip-get-the-count-of-items-in-an-enumeration/