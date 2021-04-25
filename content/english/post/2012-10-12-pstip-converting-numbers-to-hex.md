---
title: '#PSTip Converting numbers to HEX'
author: Shay Levy
type: post
date: 2012-10-12T18:00:11+00:00
url: /2012/10/12/pstip-converting-numbers-to-hex/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
PowerShell can convert Hexadecimal numbers on the command line, just prefix the value with &#8216;0x&#8217;:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; 0xAE
174
</pre>

Converting a number to HEX requires the use of .NET string formatting. Here we use the &#8216;x&#8217; format specifier, it converts a number to a string of hexadecimal digits and the result is a hexadecimal string :

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; '{0:x}' -f 174
ae
</pre>

The result is a HEX value in lower case. You can have the result in upper case by using &#8216;X&#8217; instead, and also have a specific number of digits in the result by using a precision specifier. If required, the number is padded with zeros to its left to produce the number of digits given by the precision specifier.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; '{0:X4}' -f 174
00AE
</pre>

See this page for more information on [.NET Numeric Format Strings ][1]

[1]: http://msdn.microsoft.com/en-us/library/dwhawy9k.aspx