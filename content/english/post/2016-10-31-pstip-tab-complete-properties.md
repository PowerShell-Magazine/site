---
title: '#PSTip Tab complete properties'
author: Jan Egil Ring
type: post
date: 2016-10-31T16:00:45+00:00
url: /2016/10/31/pstip-tab-complete-properties/
views:
  - 20765
post_views_count:
  - 5651
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Tab completion is a well-known feature in PowerShell, which speeds up the process of typing and reduces the risk for typing mistakes. The feature can autocomplete things like nouns and parameters, as well as values for parameters if PowerShell knows what type of object the parameter is expecting. The first two has been around since version 2.0, while the latter was introduced in version 3.0.

A less known feature is that the -Property parameter of Select-Object and the Format- cmdlets has supported parameter value completion since version 2.0.

Let us have a look at this feature in action. In the first example run in the PowerShell ISE, we press Ctrl + Space to bring up the tab completion feature:

![](/images/tab1.gif)

In the second example, we use the Tab key to invoke tab completion:

![](/images/tab2.gif)

When using the Tab key, the feature also works in the console host (powershell.exe):

![](/images/tab3.gif)

The [PSReadLine][1] module (included by default in Windows 10/WMF5 and later), also provides support for Ctrl + Space in the console host:

![](/images/tab4.gif)

[1]: https://github.com/lzybkr/PSReadLine