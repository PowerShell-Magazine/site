---
title: 'Jonathan Medd’s Favorite PowerShell Tips & Tricks'
author: Jonathan Medd
type: post
date: 2012-06-07T18:00:30+00:00
url: /2012/06/07/jonathan-medds-favorite-powershell-tips-tricks/
views:
  - 12925
post_views_count:
  - 1854
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Choose three of your favourite PowerShell tips they said. Pretty impossible to only feature three, so in the end I have picked a few that I have been working with recently.

### Alias Parameter Attribute

If you have a PowerShell function with parameters and you can’t decide on the parameter name because you potentially have multiple different matches, you can use the Alias parameter attribute to define them. Look at the following example function foo which can take some text and output it:

```
function foo {
   param (
      [Parameter(ValueFromPipelineByPropertyName=$True)]
      $text
   )

   Write-Output $text
}
```

![](/images/jmedd1.png)

It can take input from the pipeline by Property Name, i.e. if the object being passed has a property which matches the parameter name ‘text’:

![](/images/jmedd2.png)

but if there’s no match then it fails:

![](/images/jmedd3.png)

However, if we add an alias ‘name’ for the ‘text’ parameter, then we can match other input:

```
function foo {
   param (
      [Parameter(ValueFromPipelineByPropertyName=$True)]
      [Alias("Name")]
      $text
   )

   Write-Output $text
}
```

![](/images/jmedd4.png)

This can be particularly useful if you are using a ComputerName parameter, since you could create multiple aliases, e.g.:

<pre class="brush: powershell; title: ; notranslate" title="">[Alias('CN','__SERVER','IPAddress','Server','ComputerName')]</pre>

If you’re wondering why use

<pre class="brush: powershell; title: ; notranslate" title="">__SERVER</pre>

Then that’s because it is very commonly found in WMI:

![](/images/jmedd5.png)

_The next two tips are from PowerShell 3.0 RC, how they work may change by RTM._

### [pscustomobject]

In PowerShell 2.0 there are a number of different ways to create custom objects. Typically I am using these when generating report data and I want to change the labels presented for properties of the objects being returned because they don’t meet my needs.

In the following example I create a custom object of WMI data and change some of the labels. Unfortunately when it is output, I can’t control the order for the report, so I need to use Select-Object and specify the order I need.

```
function Get-Info {
   $wmiquery = Get-WmiObject -Class Win32_ComputerSystem

   New-Object psobject -Property @{
      ServerName = $wmiquery.Caption
      HypervisorType = $wmiquery.Manufacturer
      OS = (Get-WmiObject Win32_OperatingSystem).Caption
      BiosVersion = (Get-WmiObject Win32_Bios).SMBIOSBIOSVersion
      TimeZone = (Get-WmiObject Win32_Timezone).Caption
   }
}
```

![](/images/jmedd6.png)

PowerShell 3.0 introduces [pscustomobject] which I can use instead to create this custom object. Not only will I get the order I need, but typically you will see a significant performance increase too.

```
function Get-Info {
   $wmiquery = Get-WmiObject -Class Win32_ComputerSystem

   [pscustomobject] @{
      ServerName = $wmiquery.Caption
      HypervisorType = $wmiquery.Manufacturer
      OS = (Get-WmiObject Win32_OperatingSystem).Caption
      BiosVersion = (Get-WmiObject Win32_Bios).SMBIOSBIOSVersion
      TimeZone = (Get-WmiObject Win32_Timezone).Caption
   }
}
```

![](/images/jmedd7.png)

### Invoke-RestMethod

A fun one to finish off. PowerShell 3.0 introduces a new cmdlet, Invoke-RestMethod. You can use this cmdlet to interface with any REST-based API. So you might use this to manage an application which doesn’t have PowerShell cmdlets supplied by the vendor, but does supply an API.

You can use Invoke-RestMethod with the Twitter API, e.g.:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-RestMethod -Uri "http://search.twitter.com/search.json?q=PowerShell 3.0" |
Select-Object -ExpandProperty Results | Format-Table from_user,text -AutoSize
</pre>
![](/images/jmedd9.png)

If you wish to search for more than the basic set of results, you can extend the query to include multiple pages, then loop through them. I bundled this up into the below function.

```
function Get-TwitterSearch {
   [CmdletBinding()]
   param (
      [parameter(mandatory=$true)]
      [ValidateNotNullorEmpty()]
      [string]$query
   )

   $i = 1

   do {
      $uri = "http://search.twitter.com/search.json?include_entities=true&amp;result_type=recent&amp;rpp=100&amp;page=$i&amp;q=$query"
      $TwitterSearch = Invoke-RestMethod -Uri $uri
      $TwitterSearchResults = $TwitterSearchResults + $TwitterSearch.Results
      $i++
   } while ($TwitterSearch.next_page -ne $null -and $i -le 10)
   $TwitterSearchResults
}
```

![](/images/jmedd10.png)

&nbsp;