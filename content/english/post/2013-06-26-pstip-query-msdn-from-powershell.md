---
title: '# PSTip Query MSDN from PowerShell'
author: Jaap Brasser
type: post
date: 2013-06-26T18:00:40+00:00
url: /2013/06/26/pstip-query-msdn-from-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

To get more information about any object, class or attribute the MSDN library is often a good resource. Using Start-Process we can accelerate this process:

<pre class="brush: powershell; title: ; notranslate" title="">Start-Process -FilePath "http://social.msdn.microsoft.com/Search/en-US?query=Excel.Application"
</pre>

To automate this further, we can wrap this in a function.

```
Function Search-Msdn {
   param(
        [Parameter(Mandatory=$true)]
            [string[]]$SearchQuery,
        [System.Globalization.Cultureinfo]$Culture = 'en-US'
    )

	foreach ($Query in $SearchQuery) {
    	Start-Process -FilePath "http://social.msdn.microsoft.com/Search/$($Culture.Name)?query=$Query"
	}
}
```

Whenever you require more information on an object, object property or .Net class, this function will open the MSDN site displaying the search results for the query that has been entered. The full version of this function will be maintained in the Technet Script Gallery:

<http://gallery.technet.microsoft.com/Search-Msdn-a-function-eafee2bb>