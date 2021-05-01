---
title: '#PSTip Inserting characters at a specified string index position'
author: Shay Levy
type: post
date: 2013-07-04T18:00:34+00:00
url: /2013/07/04/pstip-inserting-characters-at-a-specified-string-index-position/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
How do you turn &#8216;20130701&#8217; to &#8216;2013-07-01&#8217;?

There are many ways to tackle this, breaking it into pieces (substring) and joining it back with a dash, use some regex fu to manipulate it, parse it as a DateTime and format it and so on. Here&#8217;s another way that uses the _[String.Insert][1]_ method.

The _Insert()_ method takes two arguments&#8211;the index position of the insertion and the string to insert.

```
PS> "".Insert.OverloadDefinitions
string Insert(int startIndex, string value)
```


In our example we need to add a dash after the year part which is the fourth character.

```
PS> $s = '20130701'
PS> $s = $s.Insert(4,'-')
PS> $s
2013-0701
```


Now we can add the second dash right after the month/day part (depending on your culture):

```
PS> $s = $s.Insert(7,'-')
PS> $s
2013-07-01
```


Or we can insert the dashes in two consecutive method calls:

```
PS> '20130701'.Insert(4,'-').Insert(7,'-')
2013-07-01
```


[1]: http://msdn.microsoft.com/en-us/library/system.string.insert.aspx