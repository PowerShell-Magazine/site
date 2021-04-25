---
title: Interacting with TechEd NA 2012 Schedule using PowerShell v3
author: Doug Finke
type: post
date: 2012-06-07T06:44:21+00:00
url: /2012/06/07/interacting-with-teched-na-2012-schedule-using-powershell-v3/
views:
  - 10611
post_views_count:
  - 1629
categories:
  - Module Spotlight
  - ShowUI
tags:
  - OData
  - ShowUI
  - Modules

---
The techniques used in this article are covered in deeper detail in Doug Finke’s book [“Windows PowerShell for Developers”][1] . Microsoft TechEd NA 2012 is just around corner. One way to get at content catalog is through the web page at <http://goo.gl/EMCYo>. Another way is to use PowerShell v3 and ShowUI.

Here is the PowerShell WPF GUI that lets you search the TechEd content catalog;  simple and easy, read on to see what else you can do with this PowerShell script, download it and check out how it was built.

![](/images/DougOData1.png)

### Play Along

If you want to try this out you need a few things:

  * PowerShell v3 is already a part of Windows 8 RP and Windows Server 2012 RC; for downlevel  operating systems, like Windows 7 you can download it at <http://goo.gl/x2KuA>
  * My TechEd PowerShell Scripts [https://skydrive.live.com/redir?resid=5DEC3B62D9308943!765][2]
  * ShowUI module for the custom WPF GUI <http://showui.codeplex.com/>

The only requirements are PowerShell v3 and the TechEd PowerShell scripts.

### TechEd Content at the Command Line

Here are some alternate ways to interact with the TechEd content catalog. You can run this script at the command line and search for any session with the string 2012 in it. Then pipe it to _Out-GridView_ cmdlet.

<div>
  <pre class="brush: powershell; title: ; notranslate" title="">
PS&gt; .\Get-TechEdTitle.ps1 2012 | Out-GridView
</pre>
</div>

Here is the result. You can further search/subset the list by typing in the text box at the top.

![](/images/DougOData2.png)

#### TechEd Content and Excel

Using a couple more built-in PowerShell cmdlets, I can find all the session titles that contain the word _storage_ and export the results to a comma-separated value file (CSV) and then open it in Excel, using _Invoke-Item_.

```
PS> .\Get-TechEdTitle.ps1 storage | Export-Csv .\storage.csv –NoTypeInformation
PS> Invoke-Item .\storage.csv
```

![](/images/DougOData3.png)

### How It’s Done

The interesting parts about this script are two-fold. First, the new PowerShell v3 cmdlet _Invoke-RestMethod_ is being used to retrieve the TechEd content catalog via the Internet. Second, Microsoft makes the catalog accessible using OData (Open Data Protocol) <http://www.odata.org/>. OData is an open protocol for sharing data.

> The protocol allows for a consumer to query a datasource over the HTTP protocol and get the result back in formats like Atom, JSON or plain XML, including pagination, ordering or filtering of the data.


PowerShell v3 makes it super easy to consume OData feeds whether they served either up as XML or JSON format. _Invoke-RestMethod_ auto-detects the format, consumes it and returns PowerShell objects ready for use.

One of the nice things that OData supports is the _$filter_ system query option, allowing clients to filter the data from the URL. _$filter_ specifies conditions that MUST be met by the target service for it to be returned.

```
param ([string]$Title)
$Global:baseUrl = 'http://odata.msteched.com/tena2012/sessions.svc/'
$Global:url = $baseUrl + 'Sessions'
$search = $url + "?`$filter=substringof('$($Title)', Title)"
Invoke-ODataTransform (Invoke-RestMethod $search)
```

Here is the _Invoke-ODataTransform_ function. In a nutshell here is what it does. The XML data returned from the OData query contains data types of string and _System.Xml.XmlElement_. Strings can be used as is, when you encounter a _System.Xml.XmlElement_, you need to get the _#text_ property.

Here is one way to get this done.

```
function Global:Invoke-ODataTransform ($records) {
	$propertyNames = ($records | Select -First 1).content.properties |
    	Get-Member -MemberType Properties |
    	Select -ExpandProperty name

	foreach($record in $records) {
        $h = @{}
        $h.ID = $record.ID
        $properties = $record.content.properties

        foreach($propertyName in $propertyNames) {
            $targetProperty = $properties.$propertyName
            if($targetProperty -is [System.Xml.XmlElement]) {
                $h.$propertyName = $targetProperty.'#text'
            } else {
                $h.$propertyName = $targetProperty
            }
        }
		[PSCustomObject]$h
	}
}
```

I encourage you to download and run the PowerShell script that uses ShowUI and displays the WPF GUI. ShowUI is a PowerShell module to help build WPF user interfaces in script. ShowUI makes the complicated world of WPF easy to use in PowerShell. You can use ShowUI to write simple WPF gadgets, quick front ends for your scripts, components, and full applications.

[1]: http://goo.gl/kRTSE
[2]: https://skydrive.live.com/redir?resid=5DEC3B62D9308943%21765