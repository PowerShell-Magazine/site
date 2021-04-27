---
title: '#PSTip Decoding WinRM error messages'
author: Shay Levy
type: post
date: 2013-03-06T19:00:50+00:00
url: /2013/03/06/pstip-decoding-winrm-error-messages/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When troubleshooting WinRM errors, you&#8217;ll sometime encounter hex error codes, such as 0x80338104, which don&#8217;t say much about the error itself. In such cases, you can use the _winrm_ command line tool with the _helpmsg_ parameter, to convert the error code to a readable error description.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; winrm helpmsg 0x80338104
The WS-Management service cannot process the request. The WMI service returned an 'access denied' error.
</pre>

Bonus tip: All error codes that start with 0x8033 are WinRM errors.