---
title: '#PSTip Quickly load Microsoft KB articles'
author: Shay Levy
type: post
date: 2013-02-14T23:28:36+00:00
url: /2013/02/14/pstip-quickly-load-microsoft-kb-articles/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

As system administrators, when troubleshooting an issue, we are often required to take a look at a certain knowledge base article from Microsoft. The base URL for KB articles always start with _http://support.microsoft.com/kb/_ followed by a KB id number, such as: _http://support.microsoft.com/kb/968930_.

Now, between us, will you remember that URL while working on an important issue? You&#8217;ll probably (like I do) launch your browser and rely on its address completion or use a search engine to locate it.

Instead, why not automating it with a simple one-liner function? Give it a try.

```
function kb($id) { Start-Process "http://support.microsoft.com/kb/$id" }
PS> kb 968930
```


