---
title: '#PSTip Convert PowerShell Data File to an Object'
author: Ravikanth C
type: post
date: 2016-05-11T18:00:03+00:00
url: /2016/05/11/pstip-convert-powershell-data-file-to-an-object/
views:
  - 19430
post_views_count:
  - 5970
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 4.0 or later.

Before PowerShell 4.0, if we had to convert the hash table in a PSD1 file into an object representation, we either used the _Import-LocalizedData_ cmdlet (not really meant for this purpose) or other means. For example, June Blender tweeted one such method.

{{< tweet 729774380272640000 >}}

The _Invoke-Expression_ cmdlet can be evil if you don&#8217;t know what&#8217;s inside the hash table string and can cause serious security issues. There are of course other methods such as a method I used in my [PSBookmark][1] module. I just put the hash table string in a .PS1 file and dot-source the PS1 to create an object. However, there are other better ways to achieve this.

### Using Argument Transformation

With PowerShell 4.0 we don&#8217;t need all these. For PowerShell DSC, Microsoft introduced a new attribute called _ArgumentToConfigurationDataTransformation_<span class="pl-ent">. This is used when compiling a DSC configuration. Here is how I use it.</span>

```powershell
function Get-ConfigurationDataAsObject
{
    [CmdletBinding()]
    Param (
        [Parameter(Mandatory)]
        [Microsoft.PowerShell.DesiredStateConfiguration.ArgumentToConfigurationDataTransformation()]
        [hashtable] $ConfigurationData    
    )
    return $ConfigurationData
}
```


The beauty of this method is that you can supply a .PSD1 file path and the contents (valid hash table string) gets transformed into an object.

```powershell
PS C:\> Get-ConfigurationDataAsObject -ConfigurationData C:\Documents\Github\PSBookmark\PSBookmark.psd1 -Verbose

Name                           Value
----                           -----
Copyright                      All Rights Reserved                                                   
Description                    Adds the capability to create location bookmarks for easier access.
PrivateData                    {PSData}
CompanyName                    PowerShell Magazine
GUID                           298a7c5c-d670-4093-bae2-0d6c2935a182
Author                         Ravikanth Chaganti
FunctionsToExport              {Save-LocationBookmark, Set-LocationBookmarkAsPWD, Get-LocationBookmark, Remove-LocationBookm...
VariablesToExport              *
RootModule                     PSBookmark.psm1
AliasesToExport                {goto, save, glb, rlb}
CmdletsToExport                *
ModuleVersion                  1.0.1
```

### Using AST

Converting a hashtable string to an object can be done using AST as well. The following code snippet shows that.

```powershell
$script = 'C:\documents\Github\psbookmark\psbookmark.psd1'
$scriptAST = [System.Management.Automation.Language.Parser]::ParseFile($script,[ref]$null,[ref]$null)
$hashTables = $scriptAST.FindAll({$args[0] -is [System.Management.Automation.Language.HashtableAst]}, $true)
$hashTables[0].SafeGetValue()
```


### Finally, using the Import-PowerShellDataFile cmdlet

In PowerShell 5.0, there is a new cmdlet called _Import-PowerShellDataFile_ which wraps the AST way of generating an object from the hashtable string.

```powershell
PS C:\> Import-PowerShellDataFile -Path 'C:\documents\Github\psbookmark\psbookmark.psd1'
Name                           Value 
----                           -----
Copyright                      All Rights Reserved 
Description                    Adds the capability to create location bookmarks for easier access.   
PrivateData                    {PSData}
CompanyName                    PowerShell Magazine                   
GUID                           298a7c5c-d670-4093-bae2-0d6c2935a182       
Author                         Ravikanth Chaganti
FunctionsToExport              {Save-LocationBookmark, Set-LocationBookmarkAsPWD, Get-LocationBookmark, Remove-LocationBookmark}                         
VariablesToExport              *                         
RootModule                     PSBookmark.psm1                        
AliasesToExport                {goto, save, glb, rlb}          
CmdletsToExport                *     
ModuleVersion                  1.0.1
```

&nbsp;

[1]: http://www.powershellgallery.com/packages/PSBookmark