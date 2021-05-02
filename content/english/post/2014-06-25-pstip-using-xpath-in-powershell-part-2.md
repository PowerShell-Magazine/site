---
title: '#PSTip Using XPath in PowerShell, Part 2'
author: Bartek Bielawski
type: post
date: 2014-06-25T18:00:41+00:00
url: /2014/06/25/pstip-using-xpath-in-powershell-part-2/
categories:
  - Tips and Tricks
  - XML
tags:
  - Tips and Tricks
  - XML

---
XPath can be used to apply &#8216;filter left&#8217; philosophy to XML documents. For example we can find any h1 element with &#8216;title&#8217; id using the following syntax:

<pre class="brush: powershell; title: ; notranslate" title="">Select-Xml -Path .\TestPage.xhtml -XPath "//h1[@id = 'title']"
</pre>

**Note**: You can find the code and input file <a href="https://gist.github.com/bielawb/f6b2f7a655902d3868f4" target="_blank">here</a>.

As you can see, XPath syntax has two parts: actual path (//h1) and filter (expression in square brackets). This is sufficient for simple documents. Unfortunately most XML documents used in real world scenarios contain XML namespaces. As a minimum there is a default namespace defined. Perfect example is a valid XHTML document that should have proper namespace declaration on html node:

<pre class="brush: powershell; title: ; notranslate" title="">&lt;html xmlns="http://www.w3.org/1999/xhtml"&gt;
</pre>

XML documents that define namespaces (including default namespace) require that we do the same for Select-Xml cmdlet, otherwise our XPath queries won’t work. We pass namespaces definition to Select-Xml using the –Namespace parameter. This parameter accepts a hash table with keys that we will use in our XPath queries as prefix to elements names and values equal to value of xmlns attributes:

```
Select-Xml -Path .\TestPage.xhtml -XPath "//x:h1[@id = 'title']" -Namespace @{
    x = 'http://www.w3.org/1999/xhtml'
} | ForEach-Object { $_.Node } | Format-Table -AutoSize

id    #text                                 
--    -----
title This is first H1 directly under 'body'
```

It is important to note that prefix used in XPath is owned by us; the only requirement is that is has to be unique for each namespace.