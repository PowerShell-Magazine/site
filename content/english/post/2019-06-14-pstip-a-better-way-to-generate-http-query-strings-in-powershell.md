---
title: '#PSTip A Better Way to Generate HTTP Query Strings in PowerShell'
author: Ravikanth C
type: post
date: 2019-06-14T05:41:49+00:00
url: /2019/06/14/pstip-a-better-way-to-generate-http-query-strings-in-powershell/
post_views_count:
  - 8449
views:
  - 9261
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
While working on a module that interacts with REST API, I came across a situation where I had to generate query strings. The number of parameters will vary based on what is supplied to the function. This becomes a bit tricky since the HTTP query strings have a certain format. 

For example, https://localhost:443/?name=test&age=25&company=powershell+magazine.

As you see in the above example, the first parameter in the query string should be prefixed with question mark and the subsequent parameters are separated by an ampersand. If there are spaces in a parameter value the spaces should be replaced with a plus sign. This can be coded easily in PowerShell but there is a better way using the [System.Web.HttpUtility](https://docs.microsoft.com/en-us/dotnet/api/system.web.httputility?view=netcore-2.2) class in .NET.

The ParseQueryString method in the httputility class parses the URL and gives us a key-value collection. To start with, we can provide this method an empty string.

```powershell
$nvCollection = [System.Web.HttpUtility]::ParseQueryString([String]::Empty)
```

We can then add the key value pairs to this collection.

```powershell
$nvCollection.Add('name','powershell')
$nvCollection.Add('age',13)
$nvCollection.Add('company','automate inc')
```


Once the parameters are added to the collection, we can build the URI and retrieve the query string.

```powershell
$uriRequest = [System.UriBuilder]'https://localhost'
$uriRequest.Query = $nvCollection.ToString()
$uriRequest.Uri.OriginalString
```


This is it. I created a function out of this for reuse. 

```powershell
function New-HttpQueryString
{
    [CmdletBinding()]
    param 
    (
        [Parameter(Mandatory = $true)]
        [String]
        $Uri,

        [Parameter(Mandatory = $true)]
        [Hashtable]
        $QueryParameter
    )
    # Add System.Web
    Add-Type -AssemblyName System.Web
    
    # Create a http name value collection from an empty string
    $nvCollection = [System.Web.HttpUtility]::ParseQueryString([String]::Empty)
    
    foreach ($key in $QueryParameter.Keys)
    {
        $nvCollection.Add($key, $QueryParameter.$key)
    }
    
    # Build the uri
    $uriRequest = [System.UriBuilder]$uri
    $uriRequest.Query = $nvCollection.ToString()
    
    return $uriRequest.Uri.OriginalString
}
```



[1]: https://www.headphonage.com/