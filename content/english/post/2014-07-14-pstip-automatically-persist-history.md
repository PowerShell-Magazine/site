---
title: '#PSTip Automatically persist history'
author: Jan Egil Ring
type: post
date: 2014-07-14T18:00:09+00:00
url: /2014/07/14/pstip-automatically-persist-history/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
Note: This tip requires PowerShell 2.0 or later.

By leveraging engine events and PowerShell profile scripts we can automatically persist command history to disk, and import it in the next session.

This is the code you can add to your PowerShell profile script:

```
$PSHistoryPath = Join-Path (Split-Path -Path $profile -Parent) 'PS-History.csv'
if (Test-Path -Path $PSHistoryPath) {
	Import-Csv -Path $PSHistoryPath | Add-History
}

Register-EngineEvent -SupportEvent PowerShell.Exiting -Action {
	Get-History -Count 25 | Export-Csv -Path $PSHistoryPath
}
```

The code in the action block of Register-EngineEvent will be run when PowerShell exits. New in PowerShell 4.0 and later is that the event also is triggered when you exit the PowerShell host by using “Close Window” or the “X”-button. In earlier versions the event was triggered only when using the exit command. The –SupportEvent parameter hides the subscription. Without this parameter, the subscription would be visible when running both Get-Job and Get-EventSubscriber, and we do not want to clutter the session with a profile supporting event.

In the above code we are persisting the last 25 items in the history. You can obviously edit or remove this number, but I would recommend to set a limit in order to not get a history that is too long. Remember that the history from the previous session is imported, thus over time the history will become very large if we do not configure a limit during history export.