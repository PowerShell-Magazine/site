---
title: '#PSTip Functions and SQL filtering, Part 1'
author: Bartek Bielawski
type: post
date: 2014-07-28T18:00:54+00:00
url: /2014/07/28/pstip-functions-and-sql-filtering-part-1/
categories:
  - SQL
  - Tips and Tricks
tags:
  - SQL
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or later.

This is the first tip in a series of SQL filtering tips.

I find myself using SQL as a source of the information a lot recently. My approach is always the same: I define a function that will &#8216;hide&#8217; most of the processing required and I parameterize few elements that I may want to change at the runtime. I usually define parameters for anything that will have huge impact on my SQL connection string and/or query (change database or table that I will talk to and properties that I will have available). Anything else is covered by _Filter_ parameter:

<pre class="brush: powershell; title: ; notranslate" title="">function Get-EmployeeData {
param (
    [Parameter(
        Mandatory = $true
    )]
    [string]$Filter
)
    $Query = @"
    SELECT
        FirstName, LastName, Department, Title, StartDate, EndDate
    FROM
        Employees
    WHERE
        $Filter  
"@
    $Query
}
</pre>

There are few problems with this approach. First of all you have to construct long string and quote things correctly (remember to escape single quotes surrounding values or put whole parameter in double quotes):

```
Get-EmployeeData -Filter "FirstName LIKE 'Jo%' "

 SELECT
    FirstName, LastName, Department, Title, StartDate, EndDate
FROM
    Employees
WHERE
    FirstName LIKE 'Jo%'
```

Another problems is even more annoying and is related to database schema: if you don&#8217;t know what you can filter on, you may be forced to check this schema (or function body) every time you will use your PowerShell command. To remove both problems we can define separate set of parameters that will simplify filtering on the things we use in the filter most often:

```
function Get-EmployeeData {
    [CmdletBinding(
        DefaultParameterSetName = 'filter'
    )]
    param (
        [Parameter(
            ParameterSetName = 'list'
        )]
        [string]$FirstName, 
        [Parameter(
            ParameterSetName = 'list'
        )]    
        [string]$LastName,
        [Parameter(
            ParameterSetName = 'filter',
            Mandatory = $true
        )]
        [string]$Filter
    )
        if (-not $Filter) {
            $queryList = New-Object System.Collections.ArrayList
            foreach ($key in $PSBoundParameters.Keys) {
                if ('FirstName','LastName' -contains $key) {
                    $item = "{0} LIKE '{1}'" -f $key, $PSBoundParameters[$key] 
                    $queryList.Add($item) | Out-Null
                }
            }
            $Filter = $queryList -join ' AND '
        }
        $Query = @"
        SELECT
            FirstName, LastName, Department, Title, StartDate, EndDate
        FROM
            Employees
        WHERE
            $Filter  
    "@
        $Query
}
```


Using _$PSBoundParameters_ makes building our filter relatively easy. If there is more than one element than _–join_ will put &#8216;AND&#8217; between them. With these changes we can now call our function without any quotes and we get tab-completion for any column that we use in our filters:

```
Get-EmployeeData -FirstName Jo% -LastName Do% 

 SELECT
    FirstName, LastName, Department, Title, StartDate, EndDate
FROM
    Employees
WHERE
    FirstName LIKE 'Jo%' AND LastName LIKE 'Do%'
```

This is the first step to make this function user friendly. Stay tuned for the second tip in a series.