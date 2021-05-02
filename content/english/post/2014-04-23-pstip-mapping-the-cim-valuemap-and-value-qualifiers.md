---
title: '#PSTip Mapping the CIM ValueMap and Values qualifiers'
author: Ravikanth C
type: post
date: 2014-04-23T18:00:00+00:00
url: /2014/04/23/pstip-mapping-the-cim-valuemap-and-value-qualifiers/
categories:
  - WMI
  - Tips and Tricks
tags:
  - WMI
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 3.0 or above.

At PowerShell training I did earlier this week, an attendee asked me if there is a way to map a CIM class property&#8217;s ValueMap to the Values.

The ValueMap and Values are the [CIM property qualifiers][1]. So, when looking at the CIM classes that implement such properties, we need to map the value of the property to a string in the Values qualifier for the property. For example, the [Win32_PrinterConfiguration][2] class has a property called _Color_ which defines if the printer is a Monochrome (1) or a Color (2) printer. However, the Color property has the ValueMap and Values qualifiers that define this mapping. When we get an instance of this class using the _Get-CimInstance_ cmdlet, we only see the numbers and not the mapping to string descriptions.

```
PS> Get-CimInstance -ClassName Win32_PrinterConfiguration | Select Name, Color
Name                                                                Color
----                                                                -----
Snagit 9                                                            2
Send To OneNote 2010                                                2
Microsoft XPS Document Writer                                       2
Fax                                                                 1
DESK                                                                1
Adobe PDF                                                           2
HOME 														   1
```

Now, if you want to map this to a string value for easy interpretation, you need to create a mapping between the ValueMap and Values qualifiers. I have written a generic function to get the mapping:


    Function Get-CimPropertyValueHash {
        [CmdletBinding()]
            Param (
                [Parameter()]
                [String]$Namespace='root\cimv2',
                
                [Parameter(Mandatory=$true)]
                [String]$ClassName,
    
                [Parameter(Mandatory=$true)]
                [String]$PropertyName
        )
    
         $CimProperties = Get-CimClass -Namespace $Namespace -ClassName $ClassName | Select -ExpandProperty CimClassProperties -ErrorAction SilentlyContinue
    
         if ($CimProperties) {
             $CimValueMapProperty = $CimProperties | Where-Object { ($_.Name -eq $PropertyName) -and (($_.Qualifiers -match 'ValueMap') -and ($_.Qualifiers -match 'Values') )}
    
             if ($CimValueMapProperty) {
                  $ValueMapHash = [Ordered]@{}
    
                  $Values = $CimValueMapProperty.Qualifiers["Values"].Value
                  $ValueMap = $CimValueMapProperty.Qualifiers["ValueMap"].Value
    
                  for($i = 0; $i -lt $Values.Length; $i++)
                  {
                      $ValueMapHash.Add($ValueMap[$i], $Values[$i])
                  }
                  $ValueMapHash
              } else {
                  throw "There is an error retrieving the property details. Either the property does not have the ValueMap and Values qualifiers or the PropertyName is invalid"
              }
     	} else {
          	throw "There is an error retrieving the class details. Please check the Namespace and ClassName values"
     	}
    }
It&#8217;s very easy to use this function:

```
PS> Get-CimPropertyValueHash -ClassName Win32_PrinterConfiguration -PropertyName Color
Name                 Value
----                 -----
1                    Monochrome
2                    Color
```

Once we have the mapping between the ValueMap and Values, we can use the resulting hash to get the string representation of the Color property.

```
PS> Get-CimInstance -ClassName Win32_PrinterConfiguration | Select Name,@{"Label"="ColorType";Expression={$colorHash[[string]$_.Color]}}
Name                           ColorType
----                           ---------
Snagit 9                       Color
Send To OneNote 2010           Color
Microsoft XPS Document Writer  Color
Fax                            Monochrome
DESK                           Monochrome
Adobe PDF                      Color
HOME                           Monochrome
```

Now, if you are wondering how to get the CIM class properties with the ValueMap and Values qualifiers, you can use the following code snippet:

<pre class="brush: powershell; title: ; notranslate" title="">Get-CimClass -PipelineVariable Class | Select -ExpandProperty CimClassProperties | Where-Object { $_.Qualifiers -match "ValueMap" -and $_.Qualifiers -match "Values" } | Select @{"Label"="ClassName";Expression={$Class.CimClassName}}, Name
</pre>

[1]: http://msdn.microsoft.com/en-us/library/aa393965(v=vs.85).aspx
[2]: http://msdn.microsoft.com/en-us/library/aa394364(v=vs.85).aspx