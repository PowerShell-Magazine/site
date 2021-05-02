---
title: '#PSTip How to check if a DateTime string is in a specific pattern'
author: Shay Levy
type: post
date: 2013-07-09T18:00:32+00:00
url: /2013/07/09/pstip-how-to-check-if-a-datetime-string-is-in-a-specific-pattern/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
You want to be able to check and allow only date and time strings that are in a specific date time format pattern. For example, you get a string and want to check if it is in a sortable date and time pattern.

Let&#8217;s find first how a sortable date and time string looks like. Note that format string is culture dependent:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; (Get-Culture).DateTimeFormat.SortableDateTimePattern
yyyy'-'MM'-'dd'T'HH':'mm':'ss
</pre>

Instead of using, or having to remember that long pattern, the .NET Framework offers us a composite format string

that is an equivalent of the above, kind of a shortcut pattern, the [&#8220;s&#8221; standard format string][1]. We can format a date and time object to a sortable pattern with the _Get-Date_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Date -Format s
2013-07-05T11:45:10
</pre>

Now, given a date and time string, how you check it complies to a specific pattern?

The _DateTime_ .NET structure has a static method we can use to parse strings and return _DateTime_ objects, the [_ParseExact_ method][2]. It converts the specified string representation of a date and time to its _DateTime_ equivalent. If the format of the string does not match a specified format exactly an exception is thrown.

The method accepts three arguments:

  1.  s &#8211; A string that contains a date and time to convert.
  2. format &#8211; A format specifier that defines the required format of &#8216;s&#8217;.
  3. provider &#8211; An object that supplies culture-specific format information about &#8216;s&#8217;.

Wrapping it all into a function so we can reuse it any time we need to.


    function Test-DateTimePattern
    {
        param(
            [string]$String,
            [string]$Pattern,
            [System.Globalization.CultureInfo]$Culture = (Get-Culture),
            [switch]$PassThru
        )
        $result = try{ [DateTime]::ParseExact($String,$Pattern,$Culture) } catch{}
    
        if($PassThru -and $result)
        {
            $result
        }
        else
        {
            [bool]$result
        }
    }

The function returns True/False if a given string is in the correct format. Add the _PassThru_ switch to get back the parsed date and time object if the pattern was successfully parsed.

```
PS> Test-DateTimePattern -String '12:15 PM' -Pattern t
True

# invalid pattern, missing the AM/PM designator (try Get-Date -f g)
PS> Test-DateTimePattern -String '7/5/2013 12:16' -Pattern g
False

PS> Test-DateTimePattern -String '2013-07-05T11:45:12' -Pattern s -PassThru
Friday, July 5, 2013 11:45:12 AM

# invalid pattern, pattern should be in dd/MM/yyyy
PS> Test-DateTimePattern -String '12/14/2013' -Pattern d -Culture he-IL
False
```

[1]: http://msdn.microsoft.com/en-us/library/az4se3k1.aspx
[2]: http://msdn.microsoft.com/en-us/library/w2sa9yss.aspx