---
title: Enum’s numeric values
author: Robert Robelo
type: post
date: 2011-09-13T05:27:39+00:00
excerpt: Without any loop statements or pipelines, get the numeric values of an Enum (such as IO.FileAttributes) in 1 statement.
url: /2011/09/13/enums-numeric-values/
post_views_count:
  - 1102
categories:
  - Brainteaser
tags:
  - Brainteaser

---
### Problem

Without any loop statements or pipelines, get the numeric values of an Enum (such as IO.FileAttributes) in 1 statement.

### Solution

```powershell
[Enum]::GetValues('IO.FileAttributes')
[Enum]::GetValues('IO.FileAttributes') -as 'Long[]'
```

Even though the numeric values of most Enum are of Int32, or Int (PowerShell's Int32 Type Accelerator), casting the output of Enum&#8217;s GetValues method as an Array of Int32, or Int, does not return the numeric -or integer- values. But, if the cast is to another compatible Numeric Type we get the expected output.

<p id="psmagWinner0" style="background: #B2FEE1; text-align: center;">
  Winner: James Brundage
</p>