---
title: '#PSTip Functions and SQL filtering, Part 3'
author: Bartek Bielawski
type: post
date: 2014-07-30T18:00:02+00:00
url: /2014/07/30/pstip-functions-and-sql-filtering-part-3/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 3.0 or later.

Whenever we retrieve information from SQL database we are faced with the problem: are we happy with column names defined in database schema? We may change these names relatively easily using appropriate T-SQL query, but what about parameters on our command when we do that? Will end user know how parameter name maps to object properties? Maybe it would be best to have parameter names that match properties of output object? This way filtering becomes natural.

Another problem: we don&#8217;t want to have spaces (or other special characters, for that matter) in the names of properties of our object. It would be great to rename these to something that will be practical when operating on resulting collection&#8211;grouping, sorting&#8230;

It should not be a surprise that in my opinion element in PowerShell that is best for mapping/ translating strings is a hash table. We will define such hash table that will map object properties (hash table keys) to corresponding table names (hash table values):

```
$parameterMap = @{
    GivenName = 'First Name'
    Surname = 'Last Name'
    Description = 'User Information Detailed'
    Title = 'Title'
    SamAccountName = 'Login'
    Id = 'Employee Id'
}
```


Next, we will check if any of bound parameters is present in our mapping dictionary:

```
$queryList = New-Object System.Collections.ArrayList
switch ($PSBoundParameters.Keys) {
    { $parameterMap.Keys -contains $_ } {
        $wildcard = [System.Management.Automation.WildcardPattern]$PSBoundParameters[$_]
        $wql = $wildcard.ToWql()
        $item = "[{0}] LIKE '{1}'" -f $parameterMap[$_], $wql
        $queryList.Add($item) | Out-Null
    }
}
$Filter = $queryList -join ' AND '
```


We use &#8216;safe&#8217; syntax for column names: surrounding them with square brackets prevents syntax errors caused by spaces in names. With this code we&#8217;ve managed to translate our parameter names to columns, and parameter values to patterns that can be used in SQL queries. But we also want to be sure that parameter names will match names of properties of produced object. To get there we need to build proper select statement:

```
$selectList = foreach ($key in $parameterMap.Keys) {
    "[{0}] AS '{1}'" -f $parameterMap.$key, $key
}
$select = $selectList -join ', '
```


Once we update our _Get-EmployeeData_ function with changes mentioned above we will get a tool, that works very well with _New-ADUser_ cmdlet from ActiveDirectory module. As this cmdlet accepts most of the parameters used to create new user as _ValueFromPipelineByPropertyName_, we can just pipe results from our function to it. There is one exception: we don&#8217;t have _Name_ property. But if we agree on certain pattern for that attribute, we can solve it by passing script block to this parameter:

```
Get-EmployeeData -Title *Windows* | 
    New-ADUser -Name { '{0}{1}' -f $_.GivenName, $_.Surname } -WhatIf
```

What if: Performing the operation "New" on target "CN=PaulSmith,CN=Users,DC=bielawscy,DC=com".

**Note**: You can find _Get-EmployeeData_ function <a href="https://gist.github.com/PowerShellMagazine/96a48952bb8e287d7540" target="_blank">here</a>.

Our function is very close to complete tool. One last thing that is missing is an option to use filters other than _LIKE_. Our database contains information about StartDate/EndDate that we may want to use to limit the results. We will try to address this in fourth and final part of the series. Stay tuned!