---
title: Reverse text line
author: Robert Robelo
type: post
date: 2011-09-13T03:28:00+00:00
url: /2011/09/13/reverse-text-line/
views:
  - 9137
post_views_count:
  - 1820
categories:
  - Brainteaser
tags:
  - Brainteaser

---
### Problem

Reverse a line of text without the use of Array's Reverse method, Array indices, Array notation, pipelines or loop statements.

### Solution

```powershell
$str = '!ereh saw yorliK'
-join (New-Object RegEx ., RightToLeft).Matches($str)
```

Set the RightToLeft option to an &#8220;all-characters-in-a-line&#8221; matching Regular Expression and pass the line of text to its Matches method. This will return all the characters in the line from the end to the start of the line. Then, use the Join operator to return the reversed text.


<hr style="color: #d8d8d8; height: 1px;" />

<p id="psmagWinner0" style="background: #B2FEE1; text-align: center;">
  Winner: Rob Campbell
</p>