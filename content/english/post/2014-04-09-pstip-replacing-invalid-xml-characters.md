---
title: '#PSTip Replacing invalid XML characters'
author: Shay Levy
type: post
date: 2014-04-09T18:00:25+00:00
url: /2014/04/09/pstip-replacing-invalid-xml-characters/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

When authoring XML documents, some of the data you will use in your tags are considered invalid. For example, you might want to include an ampersand character in one of the tags:

<pre class="brush: xml; title: ; notranslate" title="">&lt;Tag&gt;& $foo&lt;/Tag&gt;
</pre>
However, the & character is invalid and using it as is will generate an exception. Instead, we need to replace it with its escaped equivalent. The following table lists the characters that needs to be escaped.

| Invalid character | Replace with |
| ----------------- | ------------ |
| <                 | &lt;         |
| >                 | &gt;         |
| “                 | &quot;       |
| ‘                 | &apos;       |
| &                 | &amp;        |

Making sure a character is not invalid by looking at the value of a string is not that difficult but what if you don&#8217;t have control over the value or the content of the tag is passed via a variable? <span style="line-height: 1.5em;">This is where the <a href="http://msdn.microsoft.com/en-us/library/system.security.securityelement.escape.aspx"><em>SecurityElement.Escape</em></a> method comes into play. Similarly to the <a href="http://msdn.microsoft.com/en-us/library/system.text.regularexpressions.regex.escape.aspx"><em>Regex.Escape</em> </a>method, <em>SecurityElement.Escape </em>lets you replace invalid characters with their valid values. </span>

```
PS> $var = '& $foo'
PS> [System.Security.SecurityElement]::Escape($var)


& $foo
```


