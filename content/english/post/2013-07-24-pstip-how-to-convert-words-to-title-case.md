---
title: '#PSTip How to convert words to Title Case'
author: Shay Levy
type: post
date: 2013-07-24T18:00:40+00:00
url: /2013/07/24/pstip-how-to-convert-words-to-title-case/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Let us say that you have a list of words or a sentence that you&#8217;d like to manipulate and make sure that every word starts with a capital letter. There are many ways to do that, including string manipulation or regular expression-fu. There&#8217;s actually a way that takes into count your culture settings.

The [ToTitleCase() method][1] converts a specified string to titlecase. Here&#8217;s how the signature of the methods looks like&#8211;all it takes is a string:

```
PS> (Get-Culture).TextInfo.ToTitleCase

OverloadDefinitions
-------------------
string ToTitleCase(string str)
```

Let&#8217;s try to convert the &#8216;wAr aNd pEaCe&#8217; string:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $words = 'wAr aNd pEaCe'
PS&gt; $TextInfo = (Get-Culture).TextInfo
PS&gt; $TextInfo.ToTitleCase($words)
War And Peace
</pre>

Note that words that are entirely in upper-case are not converted.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $words = 'WAR AND PEACE'
PS&gt; $TextInfo.ToTitleCase($words)
WAR AND PEACE
</pre>

To ensure we always get back words with the first character in upper-case and the rest of the characters in lower-case, we explicitly convert the string to lower-case.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $TextInfo.ToTitleCase($words.ToLower())
War And Peace
</pre>

[1]: http://msdn.microsoft.com/en-us/library/system.globalization.textinfo.totitlecase(v=vs.80).aspx