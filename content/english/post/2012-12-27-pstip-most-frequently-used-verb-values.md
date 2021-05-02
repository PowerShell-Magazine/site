---
title: '#PSTip Most frequently used -Verb values'
author: David Moravec
type: post
date: 2012-12-27T19:00:34+00:00
url: /2012/12/27/pstip-most-frequently-used-verb-values/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In previous tip we saw how to receive all _-Verb_ values for registered extensions. But what if we want to see all available values?

We have to modify the original command by removing the _format_ operator. Further, we can also use _Group-Objec_t cmdlet to see count of these values (actions).

```
cmd /c assoc |
ForEach { $ext = ($_ -split '=')[0]; (New-Object Diagnostics.ProcessStartInfo -Argument "test$ext").Verbs } |
Group-Object |
Sort-Object Count -Desc |
Format-Table Name, Count -Auto

Name                                    Count
----                                    -----
open                                      438
shell                                     114
print                                     105
AddToPlaylistVLC                           74
PlayWithVLC                                74

 (some lines removed)

Uninstall                                   1
Author                                      1
Repair                                      1
```