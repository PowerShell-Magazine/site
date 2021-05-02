---
title: 'PSTip: Save untitled tabs in PowerShell ISE to clipboard'
author: Jaap Brasser
type: post
date: 2015-04-14T16:00:21+00:00
url: /2015/04/14/pstip-save-untitled-tabs-in-powershell-ise-to-clipboard/
views:
  - 11950
post_views_count:
  - 2244
categories:
  - Tips and Tricks
tasg:
  - Tips and Tricks
---
When working with the PowerShell ISE it can occur that there are a lot of untitled scripts left at the end of a busy day of scripting. Occasionally something interesting or useful could be left in those tabs. The following function can be used to copy the content from the untitled tabs to the clipboard.

The function uses the $_.IsUntitled property of each open file in the ISE to detect if the tab is untitled. The contents of all tabs are piped into clip.exe:

```powershell
Function Send-UntitledToClip {
    $psISE.PowerShellTabs.Files | Where-Object {$_.IsUntitled} | ForEach-Object {
        $_.Editor.Text
    } | clip.exe
} 
```