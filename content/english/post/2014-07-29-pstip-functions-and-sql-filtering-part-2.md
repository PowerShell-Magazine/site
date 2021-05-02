---
title: '#PSTip Functions and SQL filtering, Part 2'
author: Bartek Bielawski
type: post
date: 2014-07-29T18:00:38+00:00
url: /2014/07/29/pstip-functions-and-sql-filtering-part-2/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 3.0 or later.

Adding support for easy filtering on individual table columns is great, but the fact that user would have to use SQL wildcard syntax rather than wildcards that he is used to, makes it feel like a partial solution. There are two options to solve this problem: first of all, we can try to parse string passed to parameter and convert it to something that SQL would understand:

```
$Wildcard = '*Value_That_Ne?ds_some%Escaping*'

# Escape '%' and '_'
$Wildcard = $Wildcard -replace '(%|_)', '[$1]'

# Replace '*' with '%' and '?' with '_'
$Wildcard = $Wildcard -replace '\*', '%'
$Wildcard = $Wildcard -replace '\?', '_'
```


The second option requires PowerShell 3.0. To get correct results we will use _ToWql()_ method on objects of type _System.Management.Automation.WildcardPattern_:

```
$Wildcard = '*Value_That_Ne?ds_some%Escaping*'
$Wildcard = [System.Management.Automation.WildcardPattern]$Wildcard
$Wildcard.ToWql()

%Value[_]That[_]Ne_ds[_]some[%]Escaping%
```

To make it more obvious for end user that our function supports wildcards we can use another feature that was introduced in PowerShell 3.0: _[SupportsWildcard()]_ attribute. Our revised Get-EmployeeData function (for clarity with just one parameter set):

```
#requires -version 3.0
function Get-EmployeeData {
<#
    .Synopsis
    Function to get employee data from SQL database.
#>
    [CmdletBinding()]
    param (
        # Employee first name
        [SupportsWildcards()]
        [string]$FirstName,

        # Employee last name
        [SupportsWildcards()]
        [string]$LastName
    )
    
    $queryList = New-Object System.Collections.ArrayList
    foreach ($key in $PSBoundParameters.Keys) {
        if ($key -match 'FirstName|LastName') {
            $wildcard = [System.Management.Automation.WildcardPattern]$PSBoundParameters[$key]
            $wql = $wildcard.ToWql()
            $item = "{0} LIKE '{1}'" -f $key, $wql
            $queryList.Add($item) | Out-Null
        }
    }

    $Filter = $queryList -join ' AND '

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

With these changes we can use more natural syntax when looking for records in SQL source:


    Get-EmployeeData -FirstName *Jo* -LastName Do*
    SELECT
        FirstName, LastName, Department, Title, StartDate, EndDate
    FROM
        Employees
    WHERE
        FirstName LIKE '%Jo%' AND LastName LIKE 'Do%'   
It is also clear for the end user that we support this syntax as soon as he will request a help for any parameter we&#8217;ve defined with wildcards in mind:   

    Get-Help Get-EmployeeData -Parameter LastName
    
    -LastName <String>
    Employee last name
    Required?                    false
    Position?                    2
    Default value                
    Accept pipeline input?       false
    Accept wildcard characters?  true
Our function is slowly starting to look like real PowerShell tool. In the next part we will extend it to support situation when we want to name PowerShell object properties differently than SQL table columns. Stay tuned!