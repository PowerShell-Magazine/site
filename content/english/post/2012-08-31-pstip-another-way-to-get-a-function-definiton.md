---
title: '#PSTip Another way to get a function definiton'
author: Aleksandar Nikolic
type: post
date: 2012-08-31T18:00:51+00:00
url: /2012/08/31/pstip-another-way-to-get-a-function-definiton/
views:
  - 14838
post_views_count:
  - 1291
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In the previous [tip][1], we&#8217;ve showed you how to get a function definition using cool and unusual technique. Can we use the Get-Command cmdlet to get the same result? Let&#8217;s look at properties of a FunctionInfo object returned by Get-Command prompt command:

```
PS> Get-Command prompt | Get-Member
   TypeName: System.Management.Automation.FunctionInfo
Name                MemberType     Definition
----                ----------     ----------
Equals              Method         bool Equals(System.Object obj)
GetHashCode         Method         int GetHashCode()
GetType             Method         type GetType()
ResolveParameter    Method         System.Management.Automation.ParameterMetadata ResolveParameter(string name)
ToString            Method         string ToString()
PSDrive             NoteProperty   System.Management.Automation.PSDriveInfo PSDrive=Function
PSIsContainer       NoteProperty   System.Boolean PSIsContainer=False
PSPath              NoteProperty   System.String PSPath=Microsoft.PowerShell.Core\Function::prompt
PSProvider          NoteProperty   System.Management.Automation.ProviderInfo PSProvider=Microsoft.PowerShell.Core\Fu...
Capability          Property       System.Management.Automation.CommandCapability Capability {get;}
CmdletBinding       Property       bool CmdletBinding {get;}
CommandType         Property       System.Management.Automation.CommandTypes CommandType {get;}
Data                Property       System.Object Data {get;set;}
DefaultParameterSet Property       string DefaultParameterSet {get;}
Definition          Property       string Definition {get;}
Description         Property       string Description {get;set;}
HelpFile            Property       string HelpFile {get;}
Module              Property       psmoduleinfo Module {get;}
...
```

Bingo! The commands in the function are stored as a script block in the Definition property of the function.

```
PS> Get-Command prompt | Select-Object -Property Definition

Definition
----------
"PS $($executionContext.SessionState.Path.CurrentLocation)$('&gt;' * ($nestedPromptLevel + 1)) "...
```

But wait, that&#8217;s not the whole definition! Luckily, the Select-Object has the ExpandProperty parameter. As its name implies, its purpose is to expand the specified property. For example, if the specified property is an array, each value of the array is included in the output. If the property contains an object, the properties of that object are displayed in the output.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Command prompt | Select-Object -ExpandProperty Definition
"PS $($executionContext.SessionState.Path.CurrentLocation)$('&gt;' * ($nestedPromptLevel + 1)) "
# .Link
# http://go.microsoft.com/fwlink/?LinkID=225750
# .ExternalHelp System.Management.Automation.dll-help.xml
</pre>

Some people are fond of pithier syntax using dot notation:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; (Get-Command prompt).definition
</pre>

Note: If you prefer to work with the Function: drive, the following command will return the same object as Get-Command prompt:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-ChildItem function:prompt
</pre>

[1]: /2012/08/30/pstip-getting-the-content-of-an-object/