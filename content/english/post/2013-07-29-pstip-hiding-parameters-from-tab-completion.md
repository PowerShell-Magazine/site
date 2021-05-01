---
title: '#PSTip Hiding parameters from tab completion'
author: Shay Levy
type: post
date: 2013-07-29T18:00:28+00:00
url: /2013/07/29/pstip-hiding-parameters-from-tab-completion/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 4.0 or above.

When writing functions and declaring parameters, PowerShell lets us use many parameter attributes to define the behaviour of the parameter. For example, we can decide if the parameter is mandatory, positional, accepts pipeline input and more.

Starting in PowerShell 4.0 there&#8217;s a new [_ParameterAttribute_][1] member named _DontShow_ (online docs are not updated yet with that attribute).

```
PS> New-Object System.Management.Automation.ParameterAttribute

Position                        : -2147483648
ParameterSetName                : __AllParameterSets
Mandatory                       : False
ValueFromPipeline               : False
ValueFromPipelineByPropertyName : False
ValueFromRemainingArguments     : False
HelpMessage                     :
HelpMessageBaseName             :
HelpMessageResourceId           :
DontShow                        : False
TypeId                          : System.Management.Automation.ParameterAttribute
```

Let&#8217;s see how it works. We&#8217;ll create a new function with two parameters and try to hide the second one. Notice that we can use a shorthand syntax and not specify a _$true_ value. That&#8217;s true for all Boolean parameter (and _CmdletBinding_) attributes.


    function Test-DontShow
    {
        [CmdletBinding()]
        param(
            [Parameter(Mandatory)]
            [string]$Name,
    
            [Parameter(DontShow)]
            $HiddenParameter
        )
    
        $Name,$HiddenParameter
    }
Now let&#8217;s try and check which parameters are visible to tab completion:

![](/images/DontShowAttr.png)

As you can see, tab completion sees only the Name parameter, the _HiddenParameter_ parameter is not visible and is not completable. That said, if you know it&#8217;s there you can still operate it:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Test-DontShow -Name ps -HiddenParameter SortOf
ps
SortOf
</pre>

Lastly, parameters are still discoverable using _Get-Command_ regardless of their _DontShow_ value.

```
PS> (Get-Command Test-DontShow).Parameters.Keys

Name
HiddenParameter
(...)
```

I haven&#8217;t found a good reason to hide parameters yet but at least you know you can 🙂

[1]: http://msdn.microsoft.com/en-us/library/system.management.automation.parameterattribute_members(v=vs.85).aspx