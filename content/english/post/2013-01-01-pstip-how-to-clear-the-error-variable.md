---
title: '#PSTip How to clear the $error variable'
author: Aleksandar Nikolic
type: post
date: 2013-01-01T19:00:11+00:00
url: /2013/01/01/pstip-how-to-clear-the-error-variable/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 2.0 or above.

PowerShell has the _$error_ automatic variable. It contains a collection of the errors that occurred while the PowerShell engine has been running. The collection in _$error_ is an instance of _System.Collections.ArrayList_. The most recent error is the first error object in a collection&#8211;_$Error[0]_. The number of errors that are retained is controlled by the _$MaximumErrorCount_ preference variable (set to 256 by default). You can increase that number up to 32768, but that would increase the memory usage as well. Default value is usually big enough.

What to do if you want to clean out all the entries in _$error_? _$error_ is a variable, so you can try with the _Clear-Variable_ cmdlet:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Clear-Variable error -Force
Clear-Variable : Cannot overwrite variable Error because it is read-only or constant.
</pre>

Unfortunately, that doesn&#8217;t work even when you use the _-Force_ parameter.

_$error_ is also an object, so maybe the _Get-Member_ cmdlet will reveal a useful method:

```
PS> $error | Get-Member
   TypeName: System.Management.Automation.ErrorRecord
Name                  MemberType     Definition
----                  ----------     ----------
Equals                Method         bool Equals(System.Object obj)
GetHashCode           Method         int GetHashCode()
GetObjectData         Method         void GetObjectData(System.Runtime.Serialization.SerializationInfo info, System....
GetType               Method         type GetType()
ToString              Method         string ToString()
writeErrorStream      NoteProperty   System.Boolean writeErrorStream=True
CategoryInfo          Property       System.Management.Automation.ErrorCategoryInfo CategoryInfo {get;}
ErrorDetails          Property       System.Management.Automation.ErrorDetails ErrorDetails {get;set;}
Exception             Property       System.Exception Exception {get;}
FullyQualifiedErrorId Property       string FullyQualifiedErrorId {get;}
InvocationInfo        Property       System.Management.Automation.InvocationInfo InvocationInfo {get;}
PipelineIterationInfo Property       System.Collections.ObjectModel.ReadOnlyCollection[int] PipelineIterationInfo {g...
ScriptStackTrace      Property       string ScriptStackTrace {get;}
TargetObject          Property       System.Object TargetObject {get;}
PSMessageDetails      ScriptProperty System.Object PSMessageDetails {get=& { Set-StrictMode -Version 1; $this.Except...
```

This approach doesn&#8217;t work either, because you are getting information about the error objects (_System.Management.Automation.ErrorRecord_ type) contained in the _$error_ variable, not the members of the _$error_ itself. Do you remember our previous tip <a href="/2012/12/11/pstip-getting-information-about-a-collection-object-not-its-elements/" target="_blank">#PSTip Getting information about a collection object, not its elements</a>? Yes, you need to use _-InputObject_ parameter. You can easily spot the _Clear()_ method now:

```
PS> Get-Member -InputObject $error
   TypeName: System.Collections.ArrayList

Name           MemberType            Definition
----           ----------            ----------
Add            Method                int Add(System.Object value), int IList.Add(System.Object value)
AddRange       Method                void AddRange(System.Collections.ICollection c)
BinarySearch   Method                int BinarySearch(int index, int count, System.Object value, System.Collections....
Clear          Method                void Clear(), void IList.Clear()
Clone          Method                System.Object Clone(), System.Object ICloneable.Clone()
Contains       Method                bool Contains(System.Object item), bool IList.Contains(System.Object value)
...
...
```

You could clean out all the entries in _$error_ by calling _$error.Clear()_.