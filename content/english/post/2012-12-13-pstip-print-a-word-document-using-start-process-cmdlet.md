---
title: '#PSTip Print a Word document using the Start-Process cmdlet'
author: Ravikanth C
type: post
date: 2012-12-13T19:00:47+00:00
url: /2012/12/13/pstip-print-a-word-document-using-start-process-cmdlet/
post_views_count:
  - 1685
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In an [earlier tip][1], we saw how we can get the supported verbs for a given file type to use with _Start-Process_ cmdlet. In this tip, I will show you how to use these verbs in an interesting and useful way.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Start-Process -FilePath "C:\Documents\Test.Docx" -Verb Print
</pre>

Isn&#8217;t this simple? With a simple one-liner, we can print a Word document to the default print device.

[1]: /2012/12/12/pstip-explore-values-that-can-be-used-with-start-processs-verb-parameter/