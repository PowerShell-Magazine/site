---
title: '#PSTip Get a list of geographical locations'
author: Shay Levy
type: post
date: 2013-03-18T18:00:01+00:00
url: /2013/03/18/pstip-get-a-list-of-geographical-locations/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

In a [previous tip][1] I wrote about the International module. One of the commands of the module, _Get-WinHomeLocation_, returns a GeoID object which represents the home location (Country and Region) of the current user account.

```
PS> Get-WinHomeLocation
GeoId HomeLocation
----- ------------
244   United States
```

We can use the _Set-WinHomeLocation_ to change the location but it requires us to have the GeoID numeric value in advance:

<pre class="brush: powershell; title: ; notranslate" title="">Set-WinHomeLocation -GeoId &lt;int32&gt;
</pre>

Unfortunately, none of the _International_ module commands gets you a list of countries and their GeoId value. Luckily, there&#8217;s a .NET class, [_RegionaInfo_][2], you can use to get the _GeoId_ of a specific culture. The following function creates a list of _RegionalInfo_ objects for all _InstalledWin32Cultures_ cultures on the current system. Run it without any parameters and you&#8217;ll get the complete list, or supply a (partial) value.

```
function Get-RegionInfo($Name='*')
{
    $cultures = [System.Globalization.CultureInfo]::GetCultures('InstalledWin32Cultures')

	foreach($culture in $cultures)
	{
   		try{
       		$region = [System.Globalization.RegionInfo]$culture.Name

            if($region.DisplayName -like $Name)
            {
                $region
            }
   		}
   		catch {}
     }
}

PS> Get-RegionInfo -Name *isr*

Name                         : he-IL
EnglishName                  : Israel
DisplayName                  : Israel
NativeName                   : ישראל
TwoLetterISORegionName       : IL
ThreeLetterISORegionName     : ISR
ThreeLetterWindowsRegionName : ISR
IsMetric                     : True
GeoId                        : 117
CurrencyEnglishName          : Israeli New Shekel
CurrencyNativeName           : שקל חדש
CurrencySymbol               : ₪
ISOCurrencySymbol            : ILS


PS> Get-RegionInfo | Sort-Object DisplayName | Select-Object Name,DisplayName,GeoId | Out-GridView
```

![](/images/region.png)

[1]: /2013/03/13/pstip-how-to-configure-international-settings-in-powershell-3-0/ "#PSTip How to configure International settings in PowerShell 3.0"
[2]: http://msdn.microsoft.com/en-us/library/system.globalization.regioninfo(v=vs.80).aspx