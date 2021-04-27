---
title: '#PSTip Invoking methods with New-Object'
author: Shay Levy
type: post
date: 2013-03-04T19:00:12+00:00
url: /2013/03/04/pstip-invoking-methods-with-new-object/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Starting with PowerShell 2.0 we can create new objects and set property values with the _Property_ parameter. For instance, launch MS Word and set its visibility.

<pre class="brush: powershell; title: ; notranslate" title="">New-Object -ComObject Word.Application -Property @{Visible=$true}
</pre>

Did you know that you can also invoke objects methods? The next example creates a new instance of IE and navigates to a website.

<pre class="brush: powershell; title: ; notranslate" title="">$property = @{Visible=$true; Navigate2='http://PowerShellMagazine.com'}
New-Object -ComObject InternetExplorer.Application -Property $property
</pre>

In the following example we create a new _ArrayList_ object and invoke the _AddRange_ method to create 10 elements with corresponding values from 1 to 10.

<pre class="brush: powershell; title: ; notranslate" title="">New-Object System.Collections.ArrayList -Property @{AddRange = 1..10}
</pre>