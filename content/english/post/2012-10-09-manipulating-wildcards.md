---
title: Manipulating Wildcards
author: Shay Levy
type: post
date: 2012-10-09T18:00:52+00:00
url: /2012/10/09/manipulating-wildcards/
categories:
  - How To
tags:
  - How To

---
![](/images/wildcards.png)

Many Windows PowerShell cmdlets support wildcard characters for their parameter values. For example, almost every cmdlet that has the Name or Path parameter supports wildcard characters for these parameters. When writing scripts, you often need to design a function or a script that can run against a group of resources rather than against a single resource. In such cases you must provide support for wildcard characters. Inside your function you might want to know if the argument you get for a wildcard parameter actually contains wildcard characters.

**Note**: The process of using wildcard characters is sometimes referred to as _globbing_.

### WildcardPattern Static methods

The System.Management.Automation namespace has a [WildcardPattern ][1]class that represents a wildcard pattern that is used for matching. We can use its static members to escape (adding a backtick character in front of wildcard characters) or unescape wildcard patterns or check if a given string has any wildcard characters in it.

```
PS> [System.Management.Automation.WildcardPattern] | Get-Member -Static
   TypeName: System.Management.Automation.WildcardPattern

Name                       MemberType Definition
----                       ---------- ----------
ContainsWildcardCharacters Method     static bool ContainsWildcardCharacters(string pattern)
Equals                     Method     static bool Equals(System.Object objA, System.Object objB)
Escape                     Method     static string Escape(string pattern)
ReferenceEquals            Method     static bool ReferenceEquals(System.Object objA, System.Object objB)
Unescape                   Method     static string Unescape(string pattern)

#  check if a given string has any wildcard characters in it
PS> $pattern =  's?[cn]*'
PS> [System.Management.Automation.WildcardPattern]::ContainsWildcardCharacters($pattern)
True

# escape the pattern, interpret it literally
PS> [System.Management.Automation.WildcardPattern]::Escape($pattern)
s`?`[cn`]`*

# unescape

PS> [System.Management.Automation.WildcardPattern]::Unescape('s`?`[cn`]`*')
s?[cn]
```

### Creating WildcardPattern objects

The WildcardPattern class has also Instance methods. Using New-Object, we can [instantiate the class][2] with a string pattern and access the instance methods:

```
PS> New-Object System.Management.Automation.WildcardPattern -ArgumentList $pattern | Get-Member
   TypeName: System.Management.Automation.WildcardPattern

Name        MemberType Definition
----        ---------- ----------
Equals      Method     bool Equals(System.Object obj)
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
IsMatch     Method     bool IsMatch(string input)
ToString    Method     string ToString()
ToWql       Method     string ToWql()
```

With the IsMatch method we can determine if the supplied string matches the wildcard pattern you specified as an argument. The ToWql method, as its name implies, converts the pattern to a WMI Query Language (WQL) filter.

If not stated otherwise, wildcard pattern matching is performed on a case-insensitive basis. You can also create a case-insensitive pattern using <a href="http://msdn.microsoft.com/en-us/library/ms580017(v=vs.85).aspx" target="_blank">another constructor</a> by passing a <a href="http://msdn.microsoft.com/en-us/library/system.management.automation.wildcardoptions(v=vs.85).aspx" target="_blank">WilcardOptions</a> object. The following command creates a case-insensitive pattern that ignores cultural differences.

<pre class="brush: powershell; title: ; notranslate" title="">New-Object System.Management.Automation.WildcardPattern -ArgumentList  $pattern,'IgnoreCase,CultureInvariant'
</pre>

Putting it all together, you can get a string, check if it contains wildcard characters, and if so convert it to WQL (WMI Query Language) so the result can be used in a WMI filter.

Let&#8217;s see which process names match our pattern. The pattern, &#8216;s?[cn]\*&#8217;, starts with the letter &#8216;s&#8217;, then it should match any single character (e.g ?), then the letter &#8216;c&#8217; or &#8216;n&#8217;, and zero or more characters (e.g \*)

```
PS> Get-Process -Name $pattern | Format-Table Name
Name
----
svchost
svchost
svchost
svchost
svchost
svchost
svchost
svchost
svchost
svchost
svchost
SyncServer
SynTPEnh
SynTPHelper
SynTPLpr
```

As you can see all process names match the pattern. Now let&#8217;s try to convert it to WQL and use it with the Get-WmiObject cmdlet.

We&#8217;ll start by checking if the pattern contains wildcard characters, if so, the converted pattern will be written to the console, the wildcard pattern will be converted to WQL so you get all processes that match the pattern

```
$IsWP = [System.Management.Automation.WildcardPattern]::ContainsWildcardCharacters($pattern)

if($IsWP)
{
   $wp= (New-Object System.Management.Automation.WildcardPattern -ArgumentList $pattern)
   $wqlPattern = $wp.ToWql()
   Write-Host "WilcardPattern '$wildcard' in WQL is: '$wqlPattern'"
   Get-WmiObject -Class Win32_Process -Filter "Name LIKE '$wqlPattern '" | Format-Table Name
}

WilcardPattern 's?[cn]*' in WQL is: 's_[cn]%'

Name
----
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
svchost.exe
SynTPEnh.exe
SynTPLpr.exe
SynTPHelper.exe
SyncServer.exe
```

And sure enough, we get the same result!

[1]: http://msdn.microsoft.com/en-us/library/system.management.automation.wildcardpattern.aspx
[2]: http://msdn.microsoft.com/en-us/library/ms580018(v=vs.85).aspx