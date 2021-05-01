---
title: '#PSTip Using XPath in PowerShell, Part 6'
author: Bartek Bielawski
type: post
date: 2014-07-02T18:00:26+00:00
url: /2014/07/02/pstip-using-xpath-in-powershell-part-6/
categories:
  - XML
  - Tips and Tricks
tags:
  - XML
  - Tips and Tricks

---
When we analyze XML documents with XPath we may need information about number of certain items. For example, we can have an XHTML document with several tables and we can easily check which tables contain rows with &#8216;class&#8217; attribute:

```
$Common = @{
    Path = 'SomeTables.xhtml'
    Namespace = @{
        d = 'http://www.w3.org/1999/xhtml'
    }
}

Select-Xml -XPath '//d:table[d:tr[@class]]' @Common | 
    ForEach-Object { $_.Node } | Format-Table -AutoSize

id             colgroup tr                 
--             -------- --
withZebra      colgroup {tr, tr, tr, tr...}
withSmallZebra colgroup {tr, tr, tr}       
```

**Note**: You can find the input SomeTables.xhtml file <a href="https://gist.github.com/bielawb/68227e931cf81a86f9a3" target="_blank">here</a>.

We nested one query within the other so that result will be table that contain rows with &#8216;class&#8217; attribute defined. Now let&#8217;s try to get information about numbers&#8211;we may want to list only tables that have more than one row defined with &#8216;class&#8217; attribute and for that we will use count() function:

```
Select-Xml -XPath '//d:table[count(d:tr[@class]) > 1]' @Common | 
    ForEach-Object { $_.Node } | Format-Table -AutoSize

id        colgroup tr                 
--        -------- --
withZebra colgroup {tr, tr, tr, tr...}
```

And last but not least: we may use count() function few times in one query and compare the results, e.g. to get all tables where number of rows without &#8216;class&#8217; attribute is higher than number of rows with that attribute:

```
Select-Xml -XPath '//d:table[count(d:tr[@class]) < count(d:tr[not(@class)])]' @Common | 
    ForEach-Object { $_.Node } | Format-Table -AutoSize

id             colgroup tr                 
--             -------- --
withoutZebra   colgroup {tr, tr, tr, tr...}
withSmallZebra colgroup {tr, tr, tr}   
```

