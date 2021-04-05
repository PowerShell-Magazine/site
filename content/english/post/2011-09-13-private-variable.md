---
title: Private variable
author: Robert Robelo
type: post
date: 2011-09-13T05:16:32+00:00
url: /2011/09/13/private-variable/
views:
  - 6530
post_views_count:
  - 1434
categories:
  - Brainteaser
tags:
  - Brainteaser

---
#### {#psmagCategory0}

### Problem

Attain the value of a private variable declared in a child scope from that scope&#8217;s own child scope.

### Solution

```powershell
& {
    $Private:var = 7
    & {
        Get-Variable var -Scope 1 -ValueOnly
    }
}
```

Declare the private variable var in a ScritBlock's scope (let&#8217;s call it &#8220;Scope A&#8221;). Next, from the scope of an inner ScriptBlock (we'll call this one &#8220;Scope B&#8221;), call the Get-Variable Cmdlet with the appropriate arguments to retrieve only the value of the private variable that is visible only in the parent's scope (Scope &#8220;A&#8221;).


<hr style="color: #d8d8d8; height: 1px;" />

<p id="psmagWinner0" style="background: #B2FEE1; text-align: center;">
  Winner: Shay Levy
</p>