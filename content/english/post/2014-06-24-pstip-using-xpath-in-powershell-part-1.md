---
title: '#PSTip Using XPath in PowerShell, Part 1'
author: Bartek Bielawski
type: post
date: 2014-06-24T18:00:09+00:00
url: /2014/06/24/pstip-using-xpath-in-powershell-part-1/
categories:
  - Tips and Tricks
  - XML
tags:
  - Tips and Tricks
  - XML

---
This is the first tip in a series of Select-Xml/XPath tips.

Working with XML documents in PowerShell is relatively easy. For most things it is enough to read XML file and convert it to XmlDocument object using XML type accelerator:

<pre class="brush: powershell; title: ; notranslate" title="">$xml = [XML](Get-Content .\Test.xml)
</pre>

**Note**: You can find the code and input files <a href="https://gist.github.com/bielawb/7bebb72c2f34885365ae" target="_blank">here</a>.

Because PowerShell will present XML as an object with nested properties, we can easily access values of child elements:

<pre class="brush: powershell; title: ; notranslate" title="">$xml.root.node[0].subNode 
</pre>

Additionally, with features introduced in PowerShell 3.0 we can now retrieve data from any level within XML document without worrying if certain elements (like &#8216;node&#8217; in example above) will be present as an array of objects.

The moment it gets more complex is when we want to access certain type of children without knowing parent names, all the way up to root node. Or worse: children we are after can dynamically move between different parents, and we want to access their values without testing every single node within XML document.

This is when we may start to appreciate cmdlet designed to make XML parsing easier&#8211;Select-Xml. Simple example: we want to read all h1 headings within XHTML document:

```
Select-Xml -Path .\TestPage.xhtml -XPath //h1 | 
    ForEach-Object { $_.Node } | Format-Table -AutoSize InnerText, ParentNode 
InnerText                              ParentNode
---------                              ----------
This is first H1 directly under 'body' body      
This is my h1 in list!                 li        
Another h1 in table header...          th        
and h1 hidden in the table cell...     td      
```

As you can see: it worked fine for any h1 heading&#8211;one directly in the body, one in some list, in the table header, and the table cell. Getting same results using XmlDocument object and member notation would be way harder. And we haven’t even started to use things that XPath has to offer. Stay tuned for the second tip in a series.