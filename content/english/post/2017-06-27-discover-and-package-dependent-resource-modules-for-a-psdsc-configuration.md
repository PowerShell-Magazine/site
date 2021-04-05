---
title: 'Discover and Package Dependent Resource Modules for a #PSDSC Configuration'
author: Ravikanth C
type: post
date: 2017-06-27T16:00:45+00:00
url: /2017/06/27/discover-and-package-dependent-resource-modules-for-a-psdsc-configuration/
views:
  - 10320
post_views_count:
  - 4644
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
If you have ever used the <a href="https://docs.microsoft.com/en-us/powershell/module/azurerm.compute/publish-azurermvmdscconfiguration?view=azurermps-4.0.0" target="_blank" rel="noopener noreferrer">Publish-AzureRmVMDscConfiguration</a> cmdlet in the Azure PowerShell tools, you may know already that this command discovers module dependencies for a configuration and packages all dependencies along with the configuration as a zip archive.

```powershell
Publish-AzureRmVMDscConfiguration ".\MyConfiguration.ps1" -OutputArchivePath ".\MyConfiguration.ps1.zip"
```

When I first used this cmdlet, I felt this was really a good idea for on-premise build processes and immediately tried to find out how they discover module dependencies. I was almost certain that it was not just text parsing but may be a little bit more than that. This exploration lead me to the source code for this cmdlet and I certainly saw lot of traces towards <a href="https://msdn.microsoft.com/en-us/library/system.management.automation.language.ast(v=vs.85).aspx" target="_blank" rel="noopener noreferrer">AST</a> being used.

The second instance that I came across the usage of AST in finding resource module dependencies was in the Configuration function in the PSDesiredStateConfiguration module. This function, starting from WMF 5.0, has a runtime parameter called ResourceModulesTuplesToImport. 

![](/images/depend1.png)

```powershell
PS C:\> (Get-Command Configuration | Select-Object -ExpandProperty Parameters).ResourceModuleTuplesToImport

Name            : ResourceModuleTuplesToImport
ParameterType   : System.Collections.Generic.List`1[System.Tuple`3[System.String[],Microsoft.PowerShell.Commands.ModuleSpecification[],System.Version]]
ParameterSets   : {[__AllParameterSets, System.Management.Automation.ParameterSetMetadata]}
IsDynamic       : False
Aliases         : {}
Attributes      : {__AllParameterSets, System.Management.Automation.ArgumentTypeConverterAttribute}
SwitchParameter : False
```

The argument for the ResourceModulesTuplesToImport gets populated at runtime &#8212; when a Configuration gets loaded for the first time. To be specific, when you create a configuration document and load it into the memory, AST gets triggered and populates the argument to this parameter. You can trace this back to [ast.cs][1]. Here is a part of that.

```c#
///////////////////////////
// get import parameters
var bodyStatements = Body.ScriptBlock.EndBlock.Statements;
var resourceModulePairsToImport = new List<Tuple>string[], ModuleSpecification[], Version();
var resourceBody = (from stm in bodyStatements where !IsImportCommand(stm, resourceModulePairsToImport) select (StatementAst)stm.Copy()).ToList();
```


So, the whole magic of deriving the dependent modules is happening in the IsImportCommand method. Once I reviewed the code there, it wasn&#8217;t tough to reverse engineer that into PowerShell.

I published my scripts to <https://github.com/rchaganti/PSDSCUtils>. Let&#8217;s take a look at the script now.

```powershell
[CmdletBinding()]
param (
    [Parameter(Mandatory)]
    [String] $ConfigurationScript,

    [Parameter()]
    [Switch] $Package,

    [Parameter()]
    [String] $PackagePath
)

$ConfigurationScriptContent = Get-Content -Path $ConfigurationScript -Raw
$ast = [System.Management.Automation.Language.Parser]::ParseInput($ConfigurationScriptContent, [ref]$null, [ref]$null)
$configAst = $ast.FindAll({ $args[0] -is [System.Management.Automation.Language.ConfigurationDefinitionAst]}, $true)
$moduleSpecifcation = @()
foreach ($config in $configAst)
{
    $dksAst = $config.FindAll({ $args[0] -is [System.Management.Automation.Language.DynamicKeywordStatementAst]}, $true)

    foreach ($dynKeyword in $dksAst)
    {
        [System.Management.Automation.Language.CommandElementAst[]] $cea = $dynKeyword.CommandElements.Copy()
        $allCommands = [System.Management.Automation.Language.CommandAst]::new($dynKeyword.Extent, $cea, [System.Management.Automation.Language.TokenKind]::Unknown, $null)
        foreach ($importCommand in $allCommands)
        {
            if ($importCommand.CommandElements[0].Value -eq 'Import-DscResource')
            {
                [System.Management.Automation.Language.StaticBindingResult]$spBinder = [System.Management.Automation.Language.StaticParameterBinder]::BindCommand($importCommand, $false)
            
                $moduleNames = ''
                $resourceNames = ''
                $moduleVersion = ''
                foreach ($item in $spBinder.BoundParameters.GetEnumerator())
                { 
                    $parameterName = $item.key
                    $argument = $item.Value.Value.Extent.Text

                    #Check if the parametername is Name
                    $parameterToCheck = 'Name'
                    $parameterToCheckLength = $parameterToCheck.Length
                    $parameterNameLength = $parameterName.Length

                    if (($parameterNameLength -le $parameterToCheckLength) -and ($parameterName.Equals($parameterToCheck.Substring(0,$parameterNameLength))))
                    {
                        $resourceNames = $argument.Split(',')
                    }

                    #Check if the parametername is ModuleName
                    $parameterToCheck = 'ModuleName'
                    $parameterToCheckLength = $parameterToCheck.Length
                    $parameterNameLength = $parameterName.Length
                    if (($parameterNameLength -le $parameterToCheckLength) -and ($parameterName.Equals($parameterToCheck.Substring(0,$parameterNameLength))))
                    {
                        $moduleNames = $argument.Split(',')
                    }

                    #Check if the parametername is ModuleVersion
                    $parameterToCheck = 'ModuleVersion'
                    $parameterToCheckLength = $parameterToCheck.Length
                    $parameterNameLength = $parameterName.Length
                    if (($parameterNameLength -le $parameterToCheckLength) -and ($parameterName.Equals($parameterToCheck.Substring(0,$parameterNameLength))))
                    {
                        if (-not ($moduleVersion.Contains(',')))
                        {
                            $moduleVersion = $argument
                        }
                        else
                        {
                            throw 'Cannot specify more than one moduleversion' 
                        }
                    }
                }

                #Get the module details
                #"Module Names: " + $moduleNames
                #"Resource Name: " + $resourceNames
                #"Module Version: " + $moduleVersion 

                if($moduleVersion)
                {
                    if (-not $moduleNames)
                    {
                        throw '-ModuleName is required when -ModuleVersion is used'
                    }

                    if ($moduleNames.Count -gt 1)
                    {
                        throw 'Cannot specify more than one module when ModuleVersion parameter is used'
                    }
                }

                if ($resourceNames)
                {
                    if ($moduleNames.Count -gt 1)
                    {
                        throw 'Cannot specify more than one module when the Name parameter is used'
                    }
                }
            
                #We have multiple combinations of parameters possible
                #Case 1: All three are provided: ModuleName,ModuleVerison, and Name
                #Case 2: ModuleName and ModuleVersion are provided
                #Case 3: Only Name is provided
                #Case 4: Only ModuleName is provided
                
                #Case 1, 2, and 3
                #At the moment, there is no error check on the resource names supplied as argument to -Name
                if ($moduleNames)
                {
                    foreach ($module in $moduleNames)
                    {
                        if (-not ($module -eq 'PSDesiredStateConfiguration'))
                        {
                            $moduleHash = @{
                                ModuleName = $module
                            }

                            if ($moduleVersion)
                            {
                                $moduleHash.Add('ModuleVersion',$moduleVersion)
                            }
                            else
                            {
                                $availableModuleVersion = Get-RecentModuleVersion -ModuleName $module
                                $moduleHash.Add('ModuleVersion',$availableModuleVersion)
                            }

                            $moduleInfo = Get-Module -ListAvailable -FullyQualifiedName $moduleHash -Verbose:$false -ErrorAction SilentlyContinue
                            if ($moduleInfo)
                            {
                                #TODO: Check if listed resources are equal or subset of what module exports
                                $moduleSpecifcation += $moduleInfo
                            }
                            else
                            {
                                throw "No module exists with name ${module}"
                            }
                        }
                    }    
                }

                #Case 2
                #Foreach resource, we need to find a module
                if ((-not $moduleNames) -and $resourceNames)
                {
                    $moduleHash = Get-DscModulesFromResourceName -ResourceNames $resourceNames -Verbose:$false
                    foreach ($module in $moduleHash)
                    {
                        $moduleInfo = Get-Module -ListAvailable -FullyQualifiedName $module -Verbose:$false   
                        $moduleSpecifcation += $moduleInfo 
                    }
                }
            }
        }
    }
}

if ($Package)
{
    #Create a temp folder
    $null = mkdir "${env:temp}\modules" -Force -Verbose:$false

    #Copy all module folders to a temp folder
    foreach ($module in $moduleSpecifcation)
    {
        $null = mkdir "${env:temp}\modules\$($module.Name)"
        Copy-Item -Path $module.ModuleBase -Destination "${env:temp}\modules\$($module.Name)" -Container -Recurse -Verbose:$false
    }

    #Create an archive with all needed modules
    Compress-Archive -Path "${env:temp}\modules" -DestinationPath $PackagePath -Force -Verbose:$false

    #Remove the folder
    Remove-Item -Path "${env:temp}\modules" -Recurse -Force -Verbose:$false
}
else
{
    return $moduleSpecifcation
}

function Get-DscModulesFromResourceName
{
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string[]] $ResourceNames
    )

    process
    {
        $moduleInfo = Get-DscResource -Name $ResourceNames -Verbose:$false | Select -Expand ModuleName -Unique
        $moduleHash = @()
        foreach ($module in $moduleInfo)
        {
            $moduleHash += @{
                 ModuleName = $module
                 ModuleVersion = (Get-RecentModuleVersion -ModuleName $module)
            }
        }

        return $moduleHash
    }
}

function Get-DscResourcesFromModule
{
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [String] $ModuleName,

        [Parameter()]
        [Version] $ModuleVersion
    )

    process
    {
        $resourceInfo = Get-DscResource -Module $ModuleName -Verbose:$false
        if ($resourceInfo)
        {
            if ($ModuleVersion)
            {
                $resources = $resourceInfo.Where({$_.Module.Version -eq $ModuleVersion})
                return $resources.Name
            }
            else
            {
                #check if there are multiple versions of the modules; if so, return the most recent one
                $mostRecentVersion = Get-RecentModuleVersion -ModuleName $ModuleName
                Get-DscResourcesFromModule -ModuleName $ModuleName -ModuleVersion $mostRecentVersion
            }
        }
    }
}

function Get-RecentModuleVersion
{
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [String] $ModuleName
    )

    process
    {
        $moduleInfo = Get-Module -ListAvailable -Name $ModuleName -Verbose:$false | Sort -Property Version
        if ($moduleInfo)
        {
            return ($moduleInfo[-1].Version).ToString()
        }
    }
}
```

Here is how you used this script:

With just the -ConfigurationScript parameter, this script emits a ModuleInfo object that contains a list of modules that are being imported in the configuration script.

![](/images/depend2.png)

In case you need to package the modules into a zip archive, you can use the -Package and -PackagePath parameters.

```powershell
.\Get-DSCResourceModulesFromConfiguration.ps1 -ConfigurationScript C:\Scripts\VMDscDemo.ps1 -Package -PackagePath C:\Scripts\modules.zip
```

There are many uses cases for this. I use this extensively in my Hyper-V lab configurations. What are your use cases?

[1]: https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/engine/parser/ast.cs