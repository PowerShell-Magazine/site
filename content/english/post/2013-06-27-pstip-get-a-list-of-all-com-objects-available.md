---
title: '#PSTip Get a list of all Com objects available'
author: Jaap Brasser
type: post
date: 2013-06-27T18:00:11+00:00
url: /2013/06/27/pstip-get-a-list-of-all-com-objects-available/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Recently a question was posted on the PowerShell.com forums: How to get a full list of available ComObjects? This tip will show how fetch all of them from the registry.

Here is the code that we can use to generate this list:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem HKLM:\Software\Classes -ErrorAction SilentlyContinue | Where-Object {
	$_.PSChildName -match '^\w+\.\w+$' -and (Test-Path -Path "$($_.PSPath)\CLSID")
} | Select-Object -ExpandProperty PSChildName
</pre>

The first Cmdlet reads out a complete list of values from HKLM:\Software\Classes and then verifies if the following two conditions are true:

  * Does the object match the naming convention for a ComObject?
  * Does the registry key have a CLSID folder? Every registered ComObject should have a CLSID as a unique identifier.

An example of the output generated by this command is as follows:

<pre class="brush: plain; title: ; notranslate" title="">AccClientDocMgr.AccClientDocMgr
AccDictionary.AccDictionary
Access.ACCDAExtension
Access.ACCDCFile
Access.ACCDEFile
Access.ACCDTFile
Access.ACCFTFile
Access.ADEFile
</pre>

To make the process of discovering ComObject easier the following function can be used.


    function Get-ComObject {
        param(
            [Parameter(Mandatory=$true,
            ParameterSetName='FilterByName')]
            [string]$Filter,
    
            [Parameter(Mandatory=$true,
            ParameterSetName='ListAllComObjects')]
            [switch]$ListAll
        )
    
        $ListofObjects = Get-ChildItem HKLM:\Software\Classes -ErrorAction SilentlyContinue | Where-Object {
            $_.PSChildName -match '^\w+\.\w+$' -and (Test-Path -Path "$($_.PSPath)\CLSID")
        } | Select-Object -ExpandProperty PSChildName
    
        if ($Filter) {
            $ListofObjects | Where-Object {$_ -like $Filter}
        } else {
            $ListofObjects
        }
    }
This function is available in the TechNet Script Gallery:

<http://gallery.technet.microsoft.com/Get-ComObject-Function-to-50a92047>

