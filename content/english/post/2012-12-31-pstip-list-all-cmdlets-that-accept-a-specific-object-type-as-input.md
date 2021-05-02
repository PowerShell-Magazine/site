---
title: '#PSTip List all cmdlets that accept a specific object type as input'
author: Ravikanth C
type: post
date: 2012-12-31T19:00:10+00:00
url: /2012/12/31/pstip-list-all-cmdlets-that-accept-a-specific-object-type-as-input/
categories:
 - Tips and Tricks
tags:
 - Tips and Tricks
---
**Note**: This tip requires PowerShell 3.0 or above.

At times, we find the need to list all cmdlets that accept a specific type of object as input. In PowerShell 3.0, the _–ParameterType_ parameter of the _Get-Command_ cmdlet can be used to retrieve this list.

```
PS> Get-Help Get-Command -Parameter ParameterType
-ParameterType
Gets commands in the session that have parameters of the specified type. Enter the full name or partial name of a parameter type. Wildcards are supported.

The ParameterName and ParameterType parameters search only commands in the current session.
This parameter is introduced in Windows PowerShell 3.0.
```

As mentioned in the help text, the value of this parameter can be the full PS type name:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command -ParameterType System.Diagnostics.Process
</pre>

Or we can use wildcards as well – in case we don’t know the full type name:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command -ParameterType *uri*
</pre>

Or we can pass the PSTypeNames property of an object and derive the cmdlets that support the object type as input:

```
PS> Get-Command -ParameterType (((Get-Process)[0]).PSTypeNames)
CommandType     Name                                               ModuleName
-----------     ----                                               ----------
Cmdlet          Debug-Process                                      Microsoft.PowerShell.Management
Cmdlet          Get-Process                                        Microsoft.PowerShell.Management
Cmdlet          Stop-Process                                       Microsoft.PowerShell.Management
Cmdlet          Wait-Process                                       Microsoft.PowerShell.Management
```

