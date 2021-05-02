---
title: '#PSTip Swapping the value of two variables'
author: Shay Levy
type: post
date: 2013-08-07T18:00:03+00:00
url: /2013/08/07/pstip-swapping-the-value-of-two-variables/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In most programming languages the conventional way to swap the value of two variable is to use a third variable.

```
PS> $a=1
PS> $b=2

PS> $temp = $a
PS> $a = $b
PS> $b = $temp

PS> "`$a=$a,`$b=$b"
$a=2,$b=1
```

In Windows PowerShell it is much easier. We can use multiple assignments to perform the same operation in less code and without a third variable.

```
PS> $a=1
PS> $b=2
PS> $a,$b = $b,$a
PS> "`$a=$a,`$b=$b"
$a=2,$b=1
```


You can also swap multiple variables. In the following example, _$a_ will swap its value with _$c_, _$b_ will swap its value with _$d_, and so on and so forth.

```
PS> $a=1
PS> $b=2
PS> $c=3
PS> $d=4

PS> $a,$b,$d,$c = $c,$d,$a,$b
PS> "`$a=$a,`$b=$b,`$c=$c,`$d=$d"
$a=3,$b=4,$c=2,$d=1
```