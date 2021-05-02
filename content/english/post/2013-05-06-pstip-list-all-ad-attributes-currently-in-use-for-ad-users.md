---
title: '#PSTip List all AD attributes currently in use for AD users'
author: Jaap Brasser
type: post
date: 2013-05-06T18:00:00+00:00
url: /2013/05/06/pstip-list-all-ad-attributes-currently-in-use-for-ad-users/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
This question was asked on the forums recently, is it possible to list all the Active Directory attributes that are currently in use for Active Directory users. It turns out this is relatively simple to do in PowerShell.

When creating a _DirectorySearcher_ object, the default behavior is to only return the properties that have a value.

Because of this behavior we can easily list all properties available on all user objects.

By piping this output into _Group-Object_ we can get an overview of all attributes that are in use by all Active Directory users and the number of times they are used.

<pre class="brush: powershell; title: ; notranslate" title="">$Searcher = New-Object DirectoryServices.DirectorySearcher
$Searcher.Filter = '(objectcategory=user)'
$Searcher.PageSize = 500
$Searcher.FindAll() | ForEach-Object {
    $_.Properties.Keys
} | Group-Object
</pre>