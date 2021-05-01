---
title: '#PSTip Check if a folder is shared'
author: Ravikanth C
type: post
date: 2013-11-27T19:00:45+00:00
url: /2013/11/27/pstip-check-if-a-folder-is-shared/
categories:
  - WMI
  - Tips and Tricks
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 3.0 or above.

The WMI class _Win32_Share_ gives us the information about all network shares that exist on the local machine. Now, how do we check if a given folder is shared or not?

We can use the _Win32_Share_ WMI class and query for a specific path! I have put this simple logic in a function.


    Function IsFolderShared {
        [CmdletBinding()]
            param (
                [Parameter(
                    ValueFromPipeLine,
                    ValueFromPipelineByPropertyName,
                    Mandatory
                )]
                [alias("FullName")]
                [String[]]$Path
            )
            Process {
               foreach ($Folder in $Path) {
                   $WMIFolderPath = $Folder -replace '\\','\\'
                   [PSCustomObject] @{
                       "FolderPath" = $Folder
                       "IsShared" = if (Get-CimInstance -Query "SELECT * FROM Win32_Share WHERE Path='$WMIFolderPath'") { $true } else { $false }
                   }
               }
           }
      }
![](/images/isshared.png)

