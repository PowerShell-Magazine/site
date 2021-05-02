---
title: '#PSTip Fixing the output of ConvertTo-Html'
author: Shay Levy
type: post
date: 2013-03-25T18:00:59+00:00
url: /2013/03/25/pstip-fixing-the-output-of-convertto-html/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When you export PowerShell objects to HTML, using the _ConvertTo-Html_ cmdlet, and HTML links are present as the values of the object, _ConvertTo-Html_ doesn&#8217;t render them as HTML links. To illustrate, the following snippet creates a table of all cmdlet names (to save space I selected just the first 3) and their online help version.

```
$helpTbl = Get-Command -CommandType Cmdlet |
Where-Object {$_.HelpUri} |
Select Name,@{n='HelpUri';e={"<a href='$($_.HelpUri)'>$($_.HelpUri)</a>"}} -First 3 |
ConvertTo-Html

$helpTbl
```

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title>HTML TABLE</title>
</head><body>
<table>
<colgroup><col/><col/></colgroup>
<tr><th>Name</th><th>HelpUri</th></tr>
<tr><td>Add-Computer</td><td>&lt;a href=&#39;http://go.microsoft.com/fwlink/?LinkID=135194&#39;&gt;http://go.microsoft.com/fwlink/?LinkID=135194&lt;/a&gt;</td></tr>
<tr><td>Add-Content</td><td>&lt;a href=&#39;http://go.microsoft.com/fwlink/?LinkID=113278&#39;&gt;http://go.microsoft.com/fwlink/?LinkID=113278&lt;/a&gt;</td></tr>
<tr><td>Add-History</td><td>&lt;a href=&#39;http://go.microsoft.com/fwlink/?LinkID=113279&#39;&gt;http://go.microsoft.com/fwlink/?LinkID=113279&lt;/a&gt;</td></tr>
</table>
</body></html>


Notice that some characters were converted into HTML character entities. Character entities are special reserved characters in HTML. In our example, the less than (<), greater than (>), or (&#8216;) signs were converted into their respective entities so the browser can display them &#8216;as-is&#8217; and not mix them with HTML tags.

That&#8217;s nice but it defeats our purpose of making the links clickable. Here&#8217;s how the links look when exported to a web page:

![](/images/HelpUri1.png)

There is no parameter on the _ConvertTo-Html_ cmdlet to prevent the conversion from happening. To resolve that we can use the [_HtmlDecode_][1] method to convert the entities back to their HTML equivalents.

To use the _HtmlDecode_ method we first need to load the _System.Web_ assembly.

```
Add-Type -AssemblyName System.Web
[System.Web.HttpUtility]::HtmlDecode($cmd)

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"> <html xmlns="http://www.w3.org/1999/xhtml"> <head> <title>HTML TABLE</title> </head><body> <
table> <colgroup><col/><col/></colgroup> <tr><th>Name</th><th>HelpUri</th></tr> <tr><td>Add-Computer</td><td><a href='http://go.microsoft.com/fwlink/?LinkID=135194'>http://go.microsoft.com/fwlink/?LinkID
=135194</a></td></tr> <tr><td>Add-Content</td><td><a href='http://go.microsoft.com/fwlink/?LinkID=113278'>http://go.microsoft.com/fwlink/?LinkID=113278</a></td></tr> <tr><td>Add-History</td><td><a href='
http://go.microsoft.com/fwlink/?LinkID=113279'>http://go.microsoft.com/fwlink/?LinkID=113279</a></td></tr> </table> </body></html>
```


The output is not as pretty as _ConvertTo-Html_ output but it does exactly what we wanted to&#8211;no HTML entities are present. Let&#8217;s have a look of the result in a browser:

```
$helpTbl = Get-Command -CommandType Cmdlet |
Where-Object {$_.HelpUri} |
Select Name,@{n='HelpUri';e={"&lt;a href='$($_.HelpUri)'&gt;$($_.HelpUri)&lt;/a&gt;"}} -First 3 |
ConvertTo-Html

Add-Type -AssemblyName System.Web
[System.Web.HttpUtility]::HtmlDecode($helpTbl) | Out-File d:\temp\PSCommands.html
Invoke-Item d:\temp\PSCommands.html
```

![](/images/HelpUri2.png)

[1]: http://msdn.microsoft.com/en-us/library/system.web.httputility.htmldecode.aspx