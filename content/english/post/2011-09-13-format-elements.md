---
title: Format elements
author: Robert Robelo
type: post
date: 2011-09-13T05:08:41+00:00
url: /2011/09/13/format-elements/
views:
  - 4645
post_views_count:
  - 1293
categories:
  - Brainteaser
tags:
  - Brainteaser
---
### Problem

Given an Array of at least 2 elements, return a new Array of formatted elements without using array indices, pipelines or loop statements.

### Solution

```powershell
$array = 1..7
-split ('{0:C}' -f [PSObject]$array)
-split ('{0:C}' -f ($array -as 'PSObject'))
```

The secret is to cast the Array as PSObject, this causes PowerShell to enumerate all elements internally looking for extended members, the Format Operator performs its magic on each element and returns a single String with the new -formatted- String Array expanded within it. Finally the Split Operator breaks the String and we get a new String Array of formatted Strings.


<hr style="color: #d8d8d8; height: 1px;" />

<p id="psmagWinner0" style="background: #B2FEE1; text-align: center;">
  Winner: No one
</p>