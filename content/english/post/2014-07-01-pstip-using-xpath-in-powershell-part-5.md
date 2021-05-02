---
title: '#PSTip Using XPath in PowerShell, Part 5'
author: Bartek Bielawski
type: post
date: 2014-07-01T18:00:37+00:00
url: /2014/07/01/pstip-using-xpath-in-powershell-part-5/
categories:
  - XML
  - Tips and Tricks
tags:
  - XML
  - Tips and Tricks
---
XPath has one big disadvantage&#8211;it&#8217;s case sensitive. Unlike regular expressions, there is no way to &#8220;turn it off&#8221;. But there is a way around it. To make part of the query case insensitive we have to use translate() function. This function takes three arguments: string that will be modified, string that lists &#8220;originals&#8221;, and finally string that contains &#8220;replacements&#8221;. If there is more characters in the second string, then any extra character will be removed. For example, if we want any node element with attribute Name equal to &#8216;first&#8217; (case-insensitive):

```
$caseInsensitive = @'
    //node[
        translate(@Name, "FIRST", "first") = 'first'
    ]
'@

Select-Xml -XPath $caseInsensitive -Path .\Test.xml |
    ForEach-Object { $_.Node.Name }

First
```

**Note**: You can find the code and input files <a href="https://gist.github.com/bielawb/d02ef8db41d255a7c4c3" target="_blank">here</a>.

But that&#8217;s not the only situation when translate() function may come in handy. Because character than we translate to can be the same for each character replaced, we can write queries that behave almost like regular expressions. Good example would be a query to get all headings in XHTML document. Regular expression that matches node names for headings is &#8216;^h[1-6]$&#8217;. To get the same in XPath we would need to translate any number between 1 and 6 to the same character (e.g. &#8216;#&#8217;) and test if result is &#8216;h#&#8217;:

```
$almostRegex = @'
    //*[
        translate(name(), '123456', '######') = 'h#'
    ]
'@

Select-Xml -XPath $almostRegex -Path .\TestPage.xhtml | 
    Select-Object Node

Node
---- 
h1
h1
h2
h3
h6
h1
h1
```