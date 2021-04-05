---
title: '#PSTip Get last login date for local account'
author: Jaap Brasser
type: post
date: 2015-06-05T18:00:14+00:00
url: /2015/06/05/pstip-get-last-login-date-for-local-account/
views:
  - 11994
post_views_count:
  - 2780
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
Using the ADSI type accelerator in combination with WinNT provider we can retrieve the last logon time for a local account from the local SAM account database:

```powershell
([ADSI]"WinNT://computer/jaapbrasser").lastlogin
```


I created a function for this specific purpose to simply get this information from local or remote systems; the function is available in the Microsoft TechNet Script Gallery: [Get-LocalLastLogonTime][1]

```powershell
function Get-LocalLastLogonTime {
    param(
    [Parameter(
            Mandatory,
            ValueFromPipeline,
            ValueFromPipelineByPropertyName,
            Position=0
    )]
    [string[]]
        $ComputerName,
    [Parameter(
            Mandatory
    )]
    [string[]]
        $UserName
    )

    begin {
        $SelectSplat = @{
            Property = @('ComputerName','UserName','LastLogin','Error')
        }
    }

    process {
        foreach ($Computer in $ComputerName) {
            if (Test-Connection -ComputerName $Computer -Count 1 -Quiet) {
                foreach ($User in $UserName) {
                    $ObjectSplat = @{
                        ComputerName = $Computer
                        UserName = $User
                        Error = $null
                        LastLogin = $null
                    }
                    $CurrentUser = $null
                    $CurrentUser = try {([ADSI]"WinNT://$computer/$user")} catch {}
                    if ($CurrentUser.Properties.LastLogin) {
                        $ObjectSplat.LastLogin = try {
                                            [datetime](-join $CurrentUser.Properties.LastLogin)
                                        } catch {
                                            -join $CurrentUser.Properties.LastLogin
                                        }
                    } elseif ($CurrentUser.Properties.Name) {
                    } else {
                        $ObjectSplat.Error = 'User not found'
                    }
                    New-Object -TypeName PSCustomObject -Property $ObjectSplat | Select-Object @SelectSplat
                }
            } else {
                $ObjectSplat = @{
                    ComputerName = $Computer
                    Error = 'Ping failed'
                }
                New-Object -TypeName PSCustomObject -Property $ObjectSplat | Select-Object @SelectSplat
            }
        }
    }
} 
```

[1]: https://gallery.technet.microsoft.com/Get-LocalLastLogonTime-Get-b23c97c6