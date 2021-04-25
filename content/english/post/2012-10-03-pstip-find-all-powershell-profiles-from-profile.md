---
title: '#PSTip Find all PowerShell Profiles from $PROFILE'
author: Shay Levy
type: post
date: 2012-10-03T18:00:32+00:00
url: /2012/10/03/pstip-find-all-powershell-profiles-from-profile/
views:
  - 5138
post_views_count:
  - 1105
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Note: This tip requires PowerShell 2.0 and above

I have always wanted to see what all PowerShell profiles are defined and in-use on a system. Sometimes, this is very crucial to understand what exactly is causing errors in scripts.

The $PROFILE automatic variable shows the current PowerShell Profile in use within the host. Now, a less known fact is that the $PROFILE also has extended properties that show different PowerShell path properties.

Let us see how we can use this to find all PowerShell profiles in use.

<pre class="brush: powershell; title: ; notranslate" title="">function Get-PSProfile{
 $PROFILE.PSExtended.PSObject.Properties |
 Select-Object Name,Value,@{Name='IsExist';Expression={Test-Path -Path $_.Value -PathType Leaf}}
}
</pre>

This gives us an output similar to:

![](/images/Capture-copy.png)