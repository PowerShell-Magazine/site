---
title: '#PSTip Functions and SQL filtering, Part 4'
author: Bartek Bielawski
type: post
date: 2014-07-31T18:00:59+00:00
url: /2014/07/31/pstip-functions-and-sql-filtering-part-4/
categories:
  - Tips and Tricks
  - SQL
tags:
  - Tips and Tricks
  - SQL

---
**Note**: This tip requires PowerShell 3.0 or later.

Filtering information retrieved from SQL database should not be limited to text filters only. Sometimes we may want to use dates to filter out some records. Other times we may want to get only records with some numeric value higher/lower than value specified. To cover wider range of filters we will need a bit smarter logic for producing complete query.

Let&#8217;s start with dates. Our database has two fields that contain _DateTime_ value: StartDate and EndDate. Suppose we want to have switches that will cover situations when date is either in the future or in the past:


    param (
        # ... other parameters ...
        # Retrieves new-hire employees.
        [Parameter(
            ParameterSetName = 'list'
        )]
        [switch]$BeforeStartDate,
    
        # Retrieves currently hired employees.
        [Parameter(
            ParameterSetName = 'list'
        )]
        [switch]$BeforeEndDate,
    
        # Retrieves employees that were already hired.
        [Parameter(
            ParameterSetName = 'list'
        )]
        [switch]$AfterStartDate,
    
        # Retrieves laid off employees.
        [Parameter(
            ParameterSetName = 'list'
        )]
        [switch]$AfterEndDate
    )
All parameters match the same pattern: time relation (After/Before) followed by table column name. Therefore we will use _-Regex_ switch to identify either scenario. As we still want to be able to support our &#8216;translated&#8217; columns, we will need to generate appropriate pattern for that scenario too:


    $mappedParameters = '^({0})$' -f ($parameterMap.Keys -join '|')
    $queryList = New-Object System.Collections.ArrayList
    switch -Regex ($PSBoundParameters.Keys) {
        $mappedParameters {
            $wildcard = [System.Management.Automation.WildcardPattern]$PSBoundParameters[$_]
            $wql = $wildcard.ToWql()
            $item = "[{0}] LIKE '{1}'" -f $parameterMap[$_], $wql
            $queryList.Add($item) | Out-Null
        }
        ^Before {
            $field = $_ -replace '^Before'
            $item = "[{0}] &gt; '{1}'" -f $field, (Get-Date)
            $queryList.Add($item) | Out-Null
        }
    
        ^After {
            $field = $_ -replace '^After'
            $item = "[{0}] &lt; '{1}'" -f $field, (Get-Date)
            $queryList.Add($item) | Out-Null
        }
    }
There is also nothing that would prevent us from doing both things at the same time: use different filter than _LIKE_ and be able to map SQL columns to friendly parameters/properties of output object. For example: we add two parameters that enable filtering on Id property (that maps to &#8216;Employee Id&#8217; column): _IdLowerThan_ and _IdGreaterThan_. Code that would create appropriate filter for us:


    switch -Regex ($PSBoundParameters.Keys) {
        # ...
        GreaterThan$ {
            $fieldMappedTo = $_ -replace 'GreaterThan$'
            $field = $parameterMap[$fieldMappedTo]
            $item = '[{0}] &gt; {1}' -f $field, $PSBoundParameters[$_]
            $queryList.Add($item) | Out-Null
        }
        LowerThan$ {
            $fieldMappedTo = $_ -replace 'LowerThan$'
            $field = $parameterMap[$fieldMappedTo]
            $item = '[{0}] &lt; {1}' -f $field, $PSBoundParameters[$_]
            $queryList.Add($item) | Out-Null
        }
    }
Using _switch_ statement makes it easy to extend our reach for any scenario we can think of. We just need to be sure that we name our parameters in a way that makes it possible to identify all options.

**Note**: Complete function that has all the bells and whistles can be found <a href="https://gist.github.com/PowerShellMagazine/dfd48427953e9702d8e5" target="_blank">here</a>.

Example use case: find new hires that have Id lower than 7:

```
Get-EmployeeData -BeforeStartDate -IdLowerThan 7 |
    Format-Table -AutoSize

Department StartDate          EndDate            SamAccountName Description            Id Surname Title            GivenName
---------- ---------          -------            -------------- -----------            -- ------- -----            ---------
Facilities 10-Jan-15 00:00:00 04-Jan-22 00:00:00 ABeans         Security in building 4  6 Beans   Security Officer Adam   
```

 