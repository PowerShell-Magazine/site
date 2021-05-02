---
title: '#PSCXTip Testing XML for well-formedness and validity against a schema'
author: Keith Hill
type: post
date: 2012-12-19T19:00:14+00:00
url: /2012/12/19/pscxtip-testing-xml-for-well-formedness-and-validity-against-a-schema/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
We often deal with XML files whether it is modifying TFS work item templates, C# project files, or our own XML data files. When making quick changes to an XML file in a simple editor like Notepad, it is prudent to check the updated XML file before checking it back in or deploying it. The <a title="PowerShell Community Extensions" href="http://pscx.codeplex.com" target="_blank">PowerShell Community Extensions</a> (1.2 and higher) provides a command to do just that called _Test-Xml_. Using _Test-Xml_ is quite simple:

```
PS> '<doc><book/><book></doc>' | Test-Xml

WARNING: The 'book' start tag on line 1 position 14 does not match the end tag of 'doc'. Line 1, position 21.
False
```

C:\PS> ${c:data.xml} | Format-Xml -AttributesOnNewLine > data.xml

```
<?xml version="1.0" encoding="utf-8"?>
<Configuration>
</Configuration>
```

The above file is well formed e.g.:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Test-Xml .\web.config
True
</pre>

However that doesn&#8217;t mean everything is right with this file. Upon validating using a schema file provided by Visual Studio we can see that there is a problem:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Test-Xml .\web.config -Validate -SchemaPath 'C:\Program Files (x86)\Microsoft Visual Studio 11.0\Xml\Schemas\1033\DotNetConfig.xsd' –Verbose
VERBOSE: Error: The 'Configuration' element is not declared. Line 2, Position 2.
False
</pre>

As it turns out, XML element names are case-sensitive and the configuration element should have been specified like so:

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
</configuration>
```

With that slight change, we have both a well-formed and schema validated XML file e.g.:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Test-Xml .\web.config -Validate -SchemaPath 'C:\Program Files (x86)\Microsoft Visual Studio 11.0\Xml\Schemas\1033\DotNetConfig.xsd'
True
</pre>

**Note**: There are many more useful PowerShell Community Extensions (PSCX) commands. If you are interested in this great community project led by PowerShell MVPs <a title="Keith Hill's blog" href="http://rkeithhill.wordpress.com" target="_blank">Keith Hill</a> and <a title="Oisin Grehan's blog" href="http://www.nivot.org" target="_blank">Oisin Grehan</a>, give PSCX a try at <http://pscx.codeplex.com>.