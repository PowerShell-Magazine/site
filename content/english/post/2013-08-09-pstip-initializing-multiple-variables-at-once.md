---
title: '#PSTip Initializing multiple variables at once'
author: Shay Levy
type: post
date: 2013-08-09T18:00:32+00:00
url: /2013/08/09/pstip-initializing-multiple-variables-at-once/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Here&#8217;s another cool variables tip. Say you want to initialize a few variables to a specific value. Instead of doing:

```
$a = 1
$b = 1
$c = 1
$d = 1
```


Try PowerShell&#8217;s assignment expressions!

```
PS> $a=$b=$c=$d = 1
PS> $a,$b,$c,$d
1
1
1
1
```

