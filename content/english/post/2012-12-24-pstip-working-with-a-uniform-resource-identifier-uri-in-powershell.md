---
title: '#PSTip Working with a Uniform Resource Identifier (URI) in PowerShell'
author: Jaap Brasser
type: post
date: 2012-12-24T19:00:53+00:00
url: /2012/12/24/pstip-working-with-a-uniform-resource-identifier-uri-in-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

PowerShell offers a powerful method of working with URIs by leveraging the System.Uri .NET class which offers many properties and methods that provide a way to easily manipulate and compare URIs. This class can be utilized by using the _[System.Uri]_ notation in PowerShell. You can get the full list of properties and methods by using the following command:

```
PS> [System.Uri]'https://powershellmagazine.com' | Get-Member
```


An interesting feature is the ability to get the relative path of a URI. It can be done by using the _MakeRelativeUri_ method:

```
PS> $BingQuery=[System.Uri]'http://www.bing.com/search?q=powershell+magazine&go=&qs=n&form=QBLH&filt=all&pq=powershell+magazine&sc=1-18&sp=-1&sk='
PS> $BingUri=[System.Uri]'http://www.bing.com'
PS> $BingUri.MakeRelativeUri($BingQuery.AbsoluteUri).OriginalString

search?q=powershell+magazine&go=&qs=n&form=QBLH&filt=all&pq=powershell+magazine&sc=1-18&sp=-1&sk=
```

Another interesting property is the _DNSSafeHost_. It contains the DNS hostname which can be used to check connectivity as is shown in the next example:

```
PS> Test-Connection $BingQuery.DnsSafeHost
```


The _GetLeftPart()_ method can be used to only select a portion of a URI. This method takes the _[System.UriPartial]_ enum as its argument. There are four enumeration names available, which can be displayed by executing the following line of code:

```
PS> [Enum]::GetNames([System.UriPartial])
Scheme
Authority
Path
Query
```


When using _Authority_ as a partial URI only the protocol and the DNS name portion of the URI will be shown:

```
PS> $Partial = [System.UriPartial]'Authority'
PS> $BingQuery.GetLeftPart($Partial)
http://www.bing.com
```

