---
title: '#PSCXTip How to determine the byte order mark of a text file'
author: Keith Hill
type: post
date: 2012-12-17T19:00:56+00:00
url: /2012/12/17/pscxtip-how-to-determine-the-byte-order-mark-of-a-text-file/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Text files created by PowerShell are little endian Unicode (_UTF-16LE_) by default.  You can see this by inspecting the first couple of bytes of a text file for a BOM i.e. a byte order mark.  BOMs are not required but PowerShell usually create a BOM when it creates a text file.  Typical BOMs you’ll encounter with Windows and PowerShell are:

<pre class="brush: plain; title: ; notranslate" title="">UTF-8 		: 0xEF 0xBB 0xBF
UTF-16LE 	: 0xFF 0xFE
</pre>

You can’t use code like _[System.IO.File]::ReadAllText()_ to view a BOM because the bytes associated with the BOM aren’t output – just the associated text is output.  _Get-Content_ works the same way except when you use the _–Encoding Byte_ parameter.  Given a file created in PowerShell:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Date &gt; date.txt
</pre>

You can see the encoding using _Get-Content_ like so:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Content .\date.txt –Encoding Byte –TotalCount 3
255
254
13
</pre>

However, unless you’re quick with your decimal to hex conversions, this output isn’t ideal. The <a title="PowerShell Community Extensions" href="http://pscx.codeplex.com" target="_blank">PowerShell Community Extensions</a> comes with a command called _Format-Hex_ that will format its input or a specified file in hex format. This utility is much like the _od_ command from UNIX. The output from the _Format-Hex_ command for the same file as above would be:

```
PS> Format-Hex .\date.txt -Count 16
Address:  0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F ASCII
-------- ----------------------------------------------- ----------------
00000000 FF FE 0D 00 0A 00 53 00 75 00 6E 00 64 00 61 00 ......S.u.n.d.a.
```

Here we can see the first two bytes are 0x_FF 0xFE_, which is _UTF-16LE_ or little endian Unicode.  If we saved the date.txt as _UTF-8_:

```
PS> Get-Date | Out-File date.txt -Encoding Utf8
PS> Format-Hex .\date.txt -Count 16
Address:  0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F ASCII
-------- ----------------------------------------------- ----------------
00000000 EF BB BF 0D 0A 53 75 6E 64 61 79 2C 20 44 65 63 .....Sunday, Dec
```

Here we can see the _UTF-8_ BOM _0xEF 0xBB 0xBF_.  This tip is most useful when you’re processing a file created by another program with PowerShell and you need to make sure you leave the file in the same encoding that it started out with.

**Note**: There are many more useful PowerShell Community Extensions (PSCX) commands. If you are interested in this great community project led by PowerShell MVPs <a title="Keith Hill's blog" href="http://rkeithhill.wordpress.com" target="_blank">Keith Hill</a> and <a title="Oisin Grehan's blog" href="http://www.nivot.org" target="_blank">Oisin Grehan</a>, give PSCX a try at <http://pscx.codeplex.com>.