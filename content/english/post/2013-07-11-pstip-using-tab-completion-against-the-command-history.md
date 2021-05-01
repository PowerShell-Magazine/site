---
title: '#PSTip Using Tab completion against the command history'
author: Shay Levy
type: post
date: 2013-07-11T18:00:39+00:00
url: /2013/07/11/pstip-using-tab-completion-against-the-command-history/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Each time you execute a command in Windows PowerShell, the command is remembered and can be fetched by using the _Get-History_ cmdlet or by using the the up/down arrow keys.  One great, less-known, time-saver feature, allows you to quickly cycle through commands using tab completion. For example, to recall the first command containing the string &#8216;dir&#8217;, type the &#8216;#&#8217; sign followed by the pattern of the command you want to find, and then press the Tab key:

```
PS> #dir<TAB>
```


You should now see the expansion of the first command from your history that matches the pattern.

Keep pressing the TAB key to cycle through all commands that match the specified pattern.

If you know the exact command ID you can quickly retrieve it by specifying the &#8216;#&#8217; sign followed by the ID:

```
PS> #11<TAB>
```



