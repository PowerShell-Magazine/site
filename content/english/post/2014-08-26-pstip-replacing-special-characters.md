---
title: '#PSTip Replacing special characters'
author: Bartek Bielawski
type: post
date: 2014-08-26T18:00:48+00:00
url: /2014/08/26/pstip-replacing-special-characters/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Once upon a time I answered Stack Overflow question about easy way to replace &#8216;special&#8217; characters with something &#8216;web safe&#8217;. I answered question with the following code:

```
$Replacer = @{
    Å = 'aa'
    é = 'e'
}

$string_to_fix = 'æøåéüÅ'
$pattern = "[$(-join $Replacer.Keys)]"

[regex]::Replace(
    $string_to_fix, 
    $pattern, 
    { 
        $Replacer[$args[0].value] 
    }
)
```

My answer was accepted by the person asking question, but I was not fully satisfied with it. It maybe worked fine for him, but when I tried to apply the same pattern to text with Polish national characters I couldn&#8217;t get what I wanted. My problem was the fact that hash tables in PowerShell are case-insensitive. To cover both upper and lower case characters I would have to do it in two steps:

```
$lower = @{
    ą = 'a'
    ć = 'c'
    ę = 'e'
    ł = 'l'
    ń = 'n'
    ó = 'o'
    ś = 's'
    ż = 'z'
    ź = 'z'
}

$patternLower = "[$(-join $lower.Keys)]"
$lowerReplaced = [regex]::Replace(
    'Zażółć GĘŚLĄ jaźń',
    $patternLower,
    {
        $lower[$args[0].value]
    }
)

$patternUpper = $patternLower.ToUpper()

[regex]::Replace(
    $lowerReplaced,
    $patternUpper,
    {
        ($lower[$args[0].value]).ToUpper()
    }
)
Zazolc GESLA jazn
```

In the first step I use _$lower_ hash table and match any lower-case Polish character. I save result to _$lowerReplace_ variable and use it in the next Replace() call. In the second step I match upper-case Polish characters. Hash table is case-insensitive so it will return lower-case replacement. All I need to do is to convert it ToUpper().

I would prefer to do it in one step, with hash table (or any other dictionary) that is case-sensitive. I didn&#8217;t have time to investigate it further back than. But few weeks ago I came across solution for my problem. Even better &#8211; it was used in a tip that had exactly same purpose! It was in <a href="http://powershell.com/cs/blogs/tips/archive/2014/07/22/converting-special-characters-part-2.aspx" target="_blank">PowerShell.com tip about converting special characters.</a>.

With information and code provided in that tip I was able to build case-sensitive hash table. But instead of listing all letters one by one I decided to take it step further and build hash table slightly different:

```
$Replacer = New-Object hashtable
foreach ($letter in 
    Write-Output ą a Ą A ć c Ć C ę e Ę E ł l Ł L ń n Ń N ó o Ó O ś s Ś S ż z Ż Z ź z Ź Z) {
    $foreach.MoveNext() | Out-Null
    $Replacer.$letter = $foreach.Current
}
```


First, I create simple array by using the _Write-Output_ cmdlet. Each item that I want to replace is followed by a string that should be used as a replacement string. This array is processed by foreach() loop. Inside this loop I use _$foreach_ automatic variable. With _$foreach.MoveNext()_ method and _$foreach.Current_ property I can access two elements in a single cycle. Note that _$letter_ is not updated when MoveNext() method is used.

Once our case-sensitive hash table is defined we can use it in Replace() method:

```
$pattern = "[$(-join $Replacer.Keys)]"

[regex]::Replace(
    'Zażółć GĘŚLĄ jaźń',
    $pattern,
    {
        $Replacer[$args[0].value]
    }
)

Zazolc GESLA jazn
```


