---
title: '#PSTip Multiple variable assignments'
author: Shay Levy
type: post
date: 2013-08-08T18:00:44+00:00
url: /2013/08/08/pstip-multiple-variable-assignments/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In the [previous tip][1] I showed how we can use multiple assignments to swap the value of two or more variable values. In today&#8217;s tip we&#8217;ll see another cool use of multiple assignments. Consider the following variable assignment.

```
$a = 1
$b = 2
$c = 3
$d = 4
```


And with multiple assignments

```
PS> $a,$b,$c,$d = 1,2,3,4
PS> $a,$b,$c,$d
1
2
3
4
```


Big difference! Less lines of code and much more elegant. Each value of the collection on the right hand side is assigned to its corresponding variable on the left side.

We can also assign the value of an array/collection to a series of variables. Here&#8217;s an example that using the Range operator:

```
PS> $d,$e,$f,$g = 4..7
PS> $d,$e,$f,$g
4
5
6
7

# or with
PS> $one,$two,$three,$four = '1 2 3 4'.Split()
PS> $one,$two,$three,$four
1
2
3
4
```

One important thing to keep in mind is that if the collection of values is bigger than the number of variables, all remaining values will accumulate in the last variable. In the following example, variables $a, $b, and $c will contain their corresponding values (e.g 1, 2 and 3), but $d will contain multiple values: 4, 5, and 6.

```
PS> $a,$b,$c,$d = 1,2,3,4,5,6

PS> $d
4
5
6
```



[1]: /2013/08/07/pstip-swapping-the-value-of-two-variables