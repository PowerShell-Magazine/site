---
title: '#PSTip How to remove the first line from a text file'
author: Aleksandar Nikolic
type: post
date: 2012-10-08T23:07:51+00:00
url: /2012/10/08/pstip-how-to-remove-the-first-line-from-a-text-file/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
There are a lot of ways to remove the first line from a text file. I hope you will find a multiple assignment technique interesting. Here is what you need to do:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; $a,$b = Get-Content .\test.txt
PS C:\&gt; $b &gt; .\test.txt
</pre>

The first line of the test.txt file is asigned to the $a variable, the rest of the file goes to the variable $b, and then you write a content of the $b variable to the test.txt file by using > redirection operator.

But wait, there is more! You can turn that into a one-liner:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; $a, ${c:test.txt} = Get-Content .\test.txt
</pre>