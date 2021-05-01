---
title: '#PSTip Test if a URL is absolute or not'
author: Ravikanth C
type: post
date: 2014-05-29T16:17:18+00:00
url: /2014/05/29/pstip-test-if-a-url-is-absolute-or-not/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The [System.Uri][1] class in .NET provides a way to validate if a URL is an absolute URL or not. This can be quite handy when your script deals with downloading content from web pages and there is a need to validate the specified URL.

The [IsWellFormedUriString][2] method provides this capability.

```
[system.uri]::IsWellFormedUriString($url,[System.UriKind]::Absolute)
```


The [UriKind][3] enumeration has three values&#8211;Absolute, Relative, and RelativeOrAbsolute. The IsWellFormedUriString() method returns a Boolean value based on the match.

I wrapped this in a small function called Test-Url:


    Function Test-Url {
        [CmdletBinding()]
        param (
            [Parameter(Mandatory=$true)]
            [String] $Url
        )
    
        Process {
            if ([system.uri]::IsWellFormedUriString($Url,[System.UriKind]::Absolute)) {
                $true
            } else {
                $false
            }
        }
    }
And, here is a practical example where I am using this function.

```
$doc = Invoke-WebRequest -Uri 'https://www.python.org/downloads'
foreach ($href in ($doc.links.href -ne '')) {
    if (Test-Url -Url $href) {
        $href
    }
}
```


[1]: http://msdn.microsoft.com/en-us/library/vstudio/System.Uri(v=vs.110).aspx
[2]: http://msdn.microsoft.com/en-us/library/vstudio/system.uri.iswellformeduristring(v=vs.110).aspx
[3]: http://msdn.microsoft.com/en-us/library/vstudio/system.urikind(v=vs.110).aspx