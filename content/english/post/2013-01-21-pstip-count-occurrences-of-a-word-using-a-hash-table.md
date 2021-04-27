---
title: '#PSTip Count occurrences of a word using a hash table'
author: Jakub Jareš
type: post
date: 2013-01-21T19:00:49+00:00
url: /2013/01/21/pstip-count-occurrences-of-a-word-using-a-hash-table/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
On one of the PowerShell forums someone asked for help with getting the 15 most used words in a webpage. The core of the answer to that question is amazingly clever use of a hash table.

```
PS> $wordList = 'three','three','one','three','two','two'
PS> $wordStatistic = $wordList | ForEach-Object -Begin { $wordCounts=@{} } -Process { $wordCounts.$_++ } -End { $wordCounts }
PS> $wordStatistic

Name  Value
----  -----
one   1
three 3
two   2
```

The result correctly states that the word &#8216;three&#8217; occurs in the word list three times, the &#8216;two&#8217; is there two times, and the &#8216;one&#8217;, not surprisingly, once.

To understand how the trick works let’s go through it step by step. The first word in the _$wordList_ array – the word ‘three’ &#8211; is passed down the pipeline. The _$wordCounts_ hash table, created in the _Begin_ block, is queried for key named ‘three’, in our case represented by the current object in pipeline variable _$__. The value of the key value pair named ‘three’ is increased by one using the increment operator ‘++’. If the key is not present in the hash table it is automatically created. One by one the _ForEach-Object_ loop processes all the words in the array incrementing appropriate key by one on each iteration. To complete the task you simply output each key-value pair of the _$wordStatistic_ hash table using the _GetEnumerator()_ method, sort them by the _Value_ property, and select just the most used words, in our case just one.

```
$wordStatistic.GetEnumerator() |
Sort-Object -Property Value -Descending |
Select-Object -First 1

Name  Value
----  -----
three 3
```

