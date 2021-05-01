---
title: '#PSTip Reset your ISE runspace'
author: Josh Miller
type: post
date: 2013-08-05T18:00:56+00:00
url: /2013/08/05/pstip-reset-your-ise-runspace/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

I have scripts that work with multiple versions of the same third-party .NET assemblies.  Choosing between these is done when the script is run in production.  When developing and testing these scripts, I can’t load a second one after the first loads because the namespaces overlap.  If I open a new PowerShell tab, I get a new clean runspace.  No variables are brought over, and none of my assemblies are loaded.  This can also be helpful if you have setup code that works with the assembly DLL, because once you load the assembly, the file is locked.

If you wanted to have something move from the old environment to the new environment, all you would need to do is pass the information in a script block to the new PowerShell tab.


    <#
    .Synopsis
       Moves open files to a new PowerShell tab
    .Example
       Reset-IseTab –Save { function Prompt {'&gt;'}  }
    #>
    
    Function Reset-IseTab {
        Param(
            [switch]$SaveFiles,
            [ScriptBlock]$InvokeInNewTab
        )
    
        $Current = $psISE.CurrentPowerShellTab    
        $FileList = @()
    
        $Current.Files | ForEach-Object {        
            if ($SaveFiles -and (-not $_.IsSaved)) {
    
                Write-Verbose "Saving $($_.FullPath)"            
                try {
                    $_.Save()             
                    $FileList  += $_      
                } catch [System.Management.Automation.MethodInvocationException] {
                    # Save will fail saying that you need to SaveAs because the 
                    # file doesn't have a path.
                    Write-Verbose "Saving $($_.FullPath) Failed"                            
                }            
            } elseif ($_.IsSaved) {            
                $FileList  += $_
            }
        }
    
        $NewTab = $psISE.PowerShellTabs.Add() 
        $FileList | ForEach-Object { 
            $NewTab.Files.Add($_.FullPath) | Out-Null 
            $Current.Files.Remove($_) 
        }
    
        # If a code block was to be sent to the new tab, add it here. 
        #  Think module loading or dot-sourcing something to put your environment
        # correct for the specific debug session.
        if ($InvokeInNewTab) {
    
            Write-Verbose "Will call this after the Tab Loads:`n $InvokeInNewTab"
    
            # Wait for the new tab to be ready to run more commands.
            While (-not $NewTab.CanInvoke) {
                Start-Sleep -Seconds 1
            }
    
            $NewTab.Invoke($InvokeInNewTab)
        }
    
        if ($Current.Files.Count -eq 0) {        
            #Only remove the tab if all of the files closed.
            $psISE.PowerShellTabs.Remove($Current)
        }
    }
