---
title: '#PSTip Filtering a collection using comparison operators'
author: Ravikanth C
type: post
date: 2014-05-28T18:00:54+00:00
url: /2014/05/28/pstip-filtering-a-collection-using-comparison-operators/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Starting with PowerShell 3.0, it is possible to filter a collection for matching or non-matching values using comparison operators. For example, assume that I have a collection with four elements in it.

<pre class="brush: powershell; title: ; notranslate" title="">$Collection = 'test1','test2','test3','test4'
</pre>

I want to filter this collection and retrieve all values except &#8216;test3&#8217;. How do we do that? We can use comparison operators here. When using a comparison operator, when the left-hand side value is a collection, the comparison results in matching values.

<pre class="brush: powershell; title: ; notranslate" title="">$Collection -ne 'test3'
</pre>

This returns all elements except &#8216;test3&#8217;.

[Update: 5/29/2014] There are gotchas in using this method for collection filtering. [Roman Kuzmin][1] provided a great answer to a question on Stack Overflow that [explains these gotchas][2].

Here is a more practical and real-world example. I was looking for a way to filter a list of URLs for non-empty strings.

<pre class="brush: powershell; title: ; notranslate" title="">$doc = Invoke-WebRequest -Uri 'https://www.python.org/downloads'
foreach ($href in ($doc.links.href -ne '')) {
    $href
}
</pre>

Remember, the right-hand side value has to be a scalar value.

[1]: http://twitter.com/romkuzmin
[2]: http://stackoverflow.com/questions/8651905/powershell-match-operator-returns-true-but-matches-is-null/8652089#8652089