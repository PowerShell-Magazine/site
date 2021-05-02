---
title: ISE Snippets for DSC Resource functions
author: Ravikanth C
type: post
date: 2014-11-04T17:43:52+00:00
url: /2014/11/04/ise-snippets-for-dsc-resource-functions/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
PowerShell ISE 4.0 includes a couple of snippets for DSC including the DSC Resource Provider (Simple). This, in fact, is too simple.

![](/images/dscise.png)

It lacks output type for _Get_ and _Test_ functions and also the most frequently used Ensure property in the function definition. I decided to create my own snippets. I have also added the content for DSC class-defined resources in PowerShell 5.0. Here is the code for creating these snippets:

```
$GetTargetResource = @'
#TODO - Add the logic for Get-TargetResource
#TODO - Always return a hash table from this function
#TODO - Remove $Ensure if it is not required

function Get-TargetResource
{
    [OutputType([Hashtable])]
    param (
       [Parameter()]
       [ValidateSet('Present','Absent')]
       [string]
       $Ensure = 'Present'
    )
}
'@

$SetTargetResource = @'
#TODO - Add the logic for Set-TargetResource
#TODO - Do not return any value from this function
#TODO - Remove $Ensure if it is not required
function Set-TargetResource
{
    param (
       [Parameter()]
       [ValidateSet('Present','Absent')]
       [string]
       $Ensure = 'Present'
    )
}
'@

$TestTargetResource = @'
#TODO - Add the logic for Test-TargetResource
#TODO - Return only Boolean value from this function
#TODO - Remove $Ensure if it is not required
function Test-TargetResource
{
    [OutputType([boolean])]
    param (
       [parameter()]
       [ValidateSet('Present','Absent')]
       [string]
       $Ensure = 'Present'
    )
}
'@

$DSCClassResource = @'
#TODO - Update ClassName with your resource name
#TODO - Remove Ensure enum if not required
#ToDO - Add the logic required for Get, Set, and Test functions

enum Ensure
{
    Absent
    Present
}

[DscResource()]
class &lt;ClassName&gt;
{
    [Ensure] $Ensure

[void] Set()
{

}

[bool] Test()
{

}

[Hashtable] Get()
{

}

}
'@

New-IseSnippet -Title 'DSC Get-TargetResource' -Text $GetTargetResource -Description 'Get-TargetResource DSC function template' -Author 'PowerShell Magazine' -Force
New-IseSnippet -Title 'DSC Set-TargetResource' -Text $SetTargetResource -Description 'Set-TargetResource DSC function template' -Author 'PowerShell Magazine' -Force
New-IseSnippet -Title 'DSC Test-TargetResource' -Text $TestTargetResource -Description 'Test-TargetResource DSC function template' -Author 'PowerShell Magazine' -Force
New-IseSnippet -Title 'DSC Class-defined DSC resource (PowerShell 5.0)' -Text $DSCClassResource -Description 'Class-defined DSC resource template' -Author 'PowerShell Magazine' -Force
```

That creates four snippets in ISE:

![](/images/dscise2.png)

A code for these snippets is available as a [Gist on Github][1].

[1]: https://gist.github.com/rchaganti/bc7fc6e6c6509d93110f