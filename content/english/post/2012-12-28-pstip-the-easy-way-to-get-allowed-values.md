---
title: '#PSTip The easy way to get allowed values'
author: Aleksandar Nikolic
type: post
date: 2012-12-28T19:00:20+00:00
url: /2012/12/28/pstip-the-easy-way-to-get-allowed-values/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Tab completion and IntelliSense (in ISE) are greatly improved in PowerShell 3.0. They provide names of cmdlets, functions, scripts, workflows, parameters, object properties and methods, paths, files, variables, and even enumeration values. It&#8217;s very helpful to get the allowed values (this works for enumerations and ValidateSet values) when you work interactively.

But, what to do if you are still using PowerShell 2.0?

The easiest way is to try with a wrong value. An error message will give you a list of valid values.

<pre class="brush: powershell; highlight: [5]; title: ; notranslate" title="">PS&gt; Set-ExecutionPolicy -ExecutionPolicy wrongvalue
Set-ExecutionPolicy : Cannot bind parameter 'ExecutionPolicy'.
Cannot convert value "wrongvalue" to type "Microsoft.PowerShell.ExecutionPolicy" due to invalid enumeration values.
Specify one of the following enumeration values and try again.
The possible enumeration values are "Unrestricted, RemoteSigned, AllSigned, Restricted, Default, Bypass, Undefined".
</pre>

Here is another example where this approach works nicely:

```
PS> (dir test.ps1).attributes = "wrongvalue"
Exception setting "Attributes": "Cannot convert value "wrongvalue" to type "System.IO.FileAttributes" due to invalid enumeration values.
Specify one of the following enumeration values and try again.
The possible enumeration values are "ReadOnly, Hidden, System, Directory, Archive, Device, Normal, Temporary, SparseFile, ReparsePoint, Compressed, Offline, NotContentIndexed, Encrypted"."
```

