---
title: '#PSTip PowerShell 4 – Filtering a collection by using a method syntax'
author: Shay Levy
type: post
date: 2013-07-05T18:00:38+00:00
url: /2013/07/05/pstip-powershell-4-filtering-a-collection-by-using-a-method-syntax/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

One of the new features introduced in PowerShell 4.0 is the collection filtering by using a method syntax. You can now filter a collection of objects using a simplified [_Where-Object_ like syntax][1] specified as a method call.

```
PS> (Get-Process).where("name -like p*")
Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id ProcessName
-------  ------    -----      ----- -----   ------     -- -----------
465      30    86396     100996   624     2.56   9524 powershell
893      91   346044     382620  1050   515.21   9736 powershell_ise
```

Awesome! Now, if you open PowerShell and try this out you&#8217;ll probably get the following error:

<pre class="brush: powershell; title: ; notranslate" title="">PS &gt; (Get-Process).where("name -like p*")
Method invocation failed because [System.Diagnostics.Process] does not contain a method named 'where'.
At line:1 char:1
+ (gps).where("name -like p*")
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
    + FullyQualifiedErrorId : MethodNotFound
</pre>

What happened? Collection filtering is a part of the _PSDesiredStateConfiguration_ module and to have it available you first need to import the module.

```
PS> Import-Module PSDesiredStateConfiguration
PS> (Get-Process).where("name -like p*")

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id ProcessName
-------  ------    -----      ----- -----   ------     -- -----------
709      66   138168     163976   744     5.24   7120 powershell
893      91   346044     382620  1050   515.21   9736 powershell_ise

# now let's examine the method signature:
PS>  (Get-Process).where
Script              :
                               $prop, $psop, $val = [string] $args[0] -split '(
                      -eq|-ne|-gt|-ge|-lt|-le|-like|-notlike|-match|-notmatch)'
                               $operation = @{ Prop = $prop.Trim(); Value =
                      $val.Trim(); $psop = $true }
                               $this | where @operation

OverloadDefinitions : {System.Object where();}
MemberType          : ScriptMethod
TypeNameOfValue     : System.Object
Value               : System.Object where();
Name                : where
IsInstance          : False
```

As you can see the _where_ method is actually a ScriptMethod, and if you open the _PSDesiredStateConfiguration_ psm1 file you&#8217;ll find the command that extends the _System.Array_ type using the _Update-TypeData_ cmdlet (line 849).

<pre class="brush: powershell; title: ; notranslate" title="">Update-TypeData -Force -MemberType ScriptMethod -MemberName where -TypeName System.Array -Value {
         $prop, $psop, $val = [string] $args[0] -split '(-eq|-ne|-gt|-ge|-lt|-le|-like|-notlike|-match|-notmatch)'
         $operation = @{ Prop = $prop.Trim(); Value = $val.Trim(); $psop = $true }
         $this | where @operation
    }
</pre>

Having to load the module each time you want to use the method is not that convenient so why not adding it to every PowerShell session you open. Add the above command to your profile and you&#8217;re good to go.

_where()_ method is not limited to PowerShell 4.0 only. Simplified syntax was introduced in PowerShell 3.0 so you can enable this on systems with that version as well.

[1]: http://rkeithhill.wordpress.com/2011/10/19/windows-powershell-version-3-simplified-syntax/