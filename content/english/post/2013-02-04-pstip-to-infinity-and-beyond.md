---
title: '#PSTip To infinity and beyond!'
author: Jakub Jareš
type: post
date: 2013-02-04T19:00:07+00:00
url: /2013/02/04/pstip-to-infinity-and-beyond/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
They say that PowerShell is the ultimate tool that provides almost infinite number of possible applications. So I wondered: What should I do to actually express infinity in PowerShell? The answer turned out to be fairly simple; the System.Double class implements static properties that represent both positive and negative infinity.

```
PS> [System.Double]::PositiveInfinity
Infinity

PS&gt; [System.Double]::NegativeInfinity
-Infinity
```

The infinity number is defined as the result of division by zero, and we can, of course, confirm this definition:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Double]1/0
Infinity
</pre>

You probably won’t use these ‘values’ often, but you should know they can be used similarly to any other number. You can for example test if the estimated number of all atoms in the observable Universe is less than infinite:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; 10e80 –lt [System.Double]::PositiveInfinity
True
</pre>