---
title: '#PSTip Use Shell.Application to determine which folder is open in File Explorer'
author: Jaap Brasser
type: post
date: 2015-06-08T18:00:56+00:00
url: /2015/06/08/pstip-use-shell-application-to-determine-which-folder-is-open-in-file-explorer/
views:
  - 10131
post_views_count:
  - 2419
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
By using the Windows() method and the LocationURL and LocationName properties we can programmatically determine which folder is open in File Explorer.

```powershell
$ShellApp = New-Object -ComObject Shell.Application
$ShellApp.Windows() | Where-Object {$_.Name -eq 'File Explorer'} | 
Select-Object LocationName,LocationURL
```


The Where-Object statement is required to filter out results from Internet Explorer. If you are interested in all results including the open Internet Explorer windows and tabs the Where-Object statement can be omitted.