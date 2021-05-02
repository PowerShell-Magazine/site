---
title: '#PSTip Displaying all possible combinations of a flagged enumeration'
author: Ravikanth C
type: post
date: 2012-10-02T18:00:37+00:00
url: /2012/10/02/pstip-displaying-all-possible-combinations-of-a-flagged-enumeration/
views:
  - 7188
post_views_count:
  - 1353
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In the [last tip][1], I showed you how to create an enumeration object on which we can do bitwise operations to derive combinational values. Now, in this tip, let us see how to retrieve all possible combinations.

Assuming that we forgot the values assigned to each enumeration constant, let us first see how to see the values of each constant:

<pre class="brush: powershell; title: ; notranslate" title="">[Enum]::GetValues([Scripting.Skill]) | % { "{0,3} {1}" -f $([int]$_),$_ }
</pre>
So, this gives us something similar to:

![](/images/enum2.png)

Since the total of all these values is 31, let us use how to derive all possible combinations.

<pre class="brush: powershell; title: ; notranslate" title="">for ($i = 0; $i -lt 31; $i++) {
   "{0,3} {1}" -f $([int]$i), [Scripting.Skill]$([int]$i)
}
</pre>

This output is generally longer! So, try it yourself!

[1]: /2012/10/02/pstip-displaying-all-possible-combinations-of-a-flagged-enumeration/