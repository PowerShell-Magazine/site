---
title: 'PSTip: Use Shell.Application to display extended file attributes'
author: Jaap Brasser
type: post
date: 2015-04-13T18:00:37+00:00
url: /2015/04/13/pstip-use-shell-application-to-display-extended-file-attributes/
views:
  - 21582
post_views_count:
  - 8488
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
In Windows files can have a lot of additional file attributes that are not shown when using the Get-ChildItem cmdlet. To reveal these the Shell.Application can be used as it allows to retrieve these extending attributes by using the GetDetailsOf method. The following example will retrieve the first three attributes from all the files and folders in the root folder of the C:\ drive.

```powershell
$com = (New-Object -ComObject Shell.Application).NameSpace('C:\')
$com.Items() | ForEach-Object {
    New-Object -TypeName PSCustomObject -Property @{
        Name = $com.GetDetailsOf($_,0)
        Size = $com.GetDetailsOf($_,1)
        ItemType = $com.GetDetailsOf($_,2)
    }
}
```


The challenging aspect of using the Shell.Application object is that the GetDetailsOf method relies on integers as input to retrieve the additional attributes. These attributes vary by version of Windows and the localization options. For example, a Russian version of Windows not only has different names for the attributes when compared the American-English localization, but also the index numbers are not identical. To list the available attributes and their index numbers for the current localization the Get-DetailsOf method can be called against the namespace in order to retrieve the both the name  and index number for all extension attributes. The following command will list all index numbers:

```powershell
$com = (New-Object -ComObject Shell.Application).NameSpace('C:\')
for ($index = 1; $index -ne 400; $index++) {
    New-Object -TypeName PSCustomObject -Property @{
        IndexNumber = $Index
        Attribute = $com.GetDetailsOf($com,$index)
    } | Where-Object {$_.Attribute}
}
```


To work around this issue I have written a function that uses dynamic parameters to retrieve the attribute names and their corresponding index numbers to simplify the retrieval of these attributes. The full function is available in the TechNet Script Library: [Get-ExtensionAttribute][1].


```powershell
function Get-ExtensionAttribute {
    [CmdletBinding()]
    Param (
        [Parameter(ValueFromPipeline=$true,
            ValueFromPipelineByPropertyName=$true,
            Position=0)]
        [string[]]
            $FullName
    )
    DynamicParam
    {
        $Attributes = New-Object System.Management.Automation.ParameterAttribute
        $Attributes.ParameterSetName = "__AllParameterSets"
        $Attributes.Mandatory = $false
        $AttributeCollection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
        $AttributeCollection.Add($Attributes)
        $Values = @($Com=(New-Object -ComObject Shell.Application).NameSpace('C:\');1..400 | ForEach-Object {$com.GetDetailsOf($com.Items,$_)} | Where-Object {$_} | ForEach-Object {$_ -replace '\s'})
        $AttributeValues = New-Object System.Management.Automation.ValidateSetAttribute($Values)
        $AttributeCollection.Add($AttributeValues)
        $DynParam1 = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("ExtensionAttribute", [string[]], $AttributeCollection)
        $ParamDictionary = New-Object -Type System.Management.Automation.RuntimeDefinedParameterDictionary
        $ParamDictionary.Add("ExtensionAttribute", $DynParam1)
        $ParamDictionary
    }
begin {
    $ShellObject = New-Object -ComObject Shell.Application
    $DefaultName = $ShellObject.NameSpace('C:\')
    $ExtList = 0..400 | ForEach-Object {
        ($DefaultName.GetDetailsOf($DefaultName.Items,$_)).ToUpper().Replace(' ','')
    }
}

process {
    foreach ($Object in $FullName) {
        # Check if there is a fullname attribute, in case pipeline from Get-ChildItem is used
        if ($Object.FullName) {
            $Object = $Object.FullName
        }

        # Check if the path is a single file or a folder
        if (-not (Test-Path -Path $Object -PathType Container)) {
            $CurrentNameSpace = $ShellObject.NameSpace($(Split-Path -Path $Object))
            $CurrentNameSpace.Items() | Where-Object {
                $_.Path -eq $Object
            } | ForEach-Object {
                $HashProperties = @{
                    FullName = $_.Path
                }
                foreach ($Attribute in $MyInvocation.BoundParameters.ExtensionAttribute) {
                    $HashProperties.$($Attribute) = $CurrentNameSpace.GetDetailsOf($_,$($ExtList.IndexOf($Attribute.ToUpper())))
                }
                New-Object -TypeName PSCustomObject -Property $HashProperties
            }
        } elseif (-not $input) {
            $CurrentNameSpace = $ShellObject.NameSpace($Object)
            $CurrentNameSpace.Items() | ForEach-Object {
                $HashProperties = @{
                    FullName = $_.Path
                }
                foreach ($Attribute in $MyInvocation.BoundParameters.ExtensionAttribute) {
                    $HashProperties.$($Attribute) = $CurrentNameSpace.GetDetailsOf($_,$($ExtList.IndexOf($Attribute.ToUpper())))
                }
                New-Object -TypeName PSCustomObject -Property $HashProperties
            }
        }
    }
}

end {
    Remove-Variable -Force -Name DefaultName
    Remove-Variable -Force -Name CurrentNameSpace
    Remove-Variable -Force -Name ShellObject
}
}
```
[1]: https://gallery.technet.microsoft.com/Get-ExtensionAttribute-ff7dc182