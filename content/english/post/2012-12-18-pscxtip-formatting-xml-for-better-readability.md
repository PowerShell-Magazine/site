---
title: '#PSCXTip Formatting XML for better readability'
author: Keith Hill
type: post
date: 2012-12-18T19:00:53+00:00
url: /2012/12/18/pscxtip-formatting-xml-for-better-readability/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
XML is great for easily storing structured data and is pretty easy to work with in PowerShell. However, sometimes you run across XML that is so poorly formatted you lose out on XML’s human readability. Here’s a simple example – the entire set of XML on a single line:

<pre class="brush: powershell; title: ; notranslate" title="">&lt;AFX_RIBBON attribute1="foo" attribute2="bar"&gt;&lt;HEADER&gt;&lt;VERSION&gt;1&lt;/VERSION&gt;&lt;/HEADER&gt;&lt;RIBBON_BAR&gt;&lt;ELEMENT_NAME&gt;RibbonBar&lt;/ELEMENT_NAME&gt;&lt;/RIBBON_BAR&gt;&lt;/AFX_RIBBON&gt;
</pre>

The space for the column is short so I’ve shown an example that isn’t too abusive. I’ve run across single line XML files that were thousands of characters wide. When you run into these situations, you’d really like a tool for pretty-print the XML for you. The <a href="http://pscx.codeplex.com" target="_blank">PowerShell Community Extensions</a> (1.2 or higher) comes with such a command – _Format-Xml_. Here’s an example of its usage based on the XML shown above:

```
PS> Format-Xml -InputObject $xml
<AFX_RIBBON attribute1="foo" attribute2="bar">
  <HEADER>
    <VERSION>1</VERSION>
  </HEADER>
  <RIBBON_BAR>
    <ELEMENT_NAME>RibbonBar</ELEMENT_NAME>
  </RIBBON_BAR>
</AFX_RIBBON>
```

_Format-Xml_ has a few useful options like allowing you to specify that attributes should appear on new lines e.g.:

```
PS> Format-Xml -InputObject $xml –AttributesOnNewLine
<AFX_RIBBON
  attribute1="foo"
  attribute2="bar">
  <HEADER>
    <VERSION>1</VERSION>
  </HEADER>
  <RIBBON_BAR>
    <ELEMENT_NAME>RibbonBar</ELEMENT_NAME>
  </RIBBON_BAR>
</AFX_RIBBON>
```

There are also options for configuring the IndentString, setting the XML conformance level and omitting the XML declaration. Keep in mind you can load an XML, reformat it using _Format-Xml_ and save it back out e.g.:

C:\PS> ${c:data.xml} | Format-Xml -AttributesOnNewLine > data.xml

The above trick requires that data.xml be in the current directory. Still it is a handy way to read & write the same file in a one-liner.

**Note**: There are many more useful PowerShell Community Extensions (PSCX) commands. If you are interested in this great community project led by PowerShell MVPs <a title="Keith Hill's blog" href="http://rkeithhill.wordpress.com" target="_blank">Keith Hill</a> and <a title="Oisin Grehan's blog" href="http://www.nivot.org" target="_blank">Oisin Grehan</a>, give PSCX a try at <http://pscx.codeplex.com>.