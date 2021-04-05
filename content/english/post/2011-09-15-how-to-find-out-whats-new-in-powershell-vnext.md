---
title: How to find out what’s new in PowerShell vNext
author: Shay Levy
type: post
date: 2011-09-14T19:49:50+00:00
url: /2011/09/15/how-to-find-out-whats-new-in-powershell-vnext/
post_views_count:
  - 3244
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Windows 8 Developer Preview (pre-beta version) was released for public at the [BUILD conference][1] (you can download it [HERE][2]) and it is packaged with PowerShell v3 and a ton of goodies! One aspect that I usually check when a new version is released is which new cmdlets were added and what parameters has been added or removed.

To find all the changes between the two versions of PowerShell (v2 and v3) I put together a script that produces a list of commands and parameters. To compare the changes I run a command, on each version of PowerShell , that exports all core cmdlets (cmdlet name and a list of parameters) to an XML file using the Export-CliXml cmdlet.

Once the XML files have been created I import them back using the Import-CliXml cmdlet and then look at the differences. The script can be used, with some modifications, to find differences in other areas too- aliases, automatic and environment variables, specific module cmdlets and so on. Take the time to experiment with it.

<span style="text-decoration: underline;"><strong>UPDATE</strong></span>

There seems to be an issue with the command I used to export cmdlets and parameters in Windows 8 (works fine in v2).

For some reason, some cmdlets do not have a value in the Parameters column.

To resolve this I modified the expression to get the parameters and added a second call to Get-Command.

The new expression I used (post code is updated as well as the result).  @mjolinor, thanks :for the comment!

```powershell
Get-Command -Module Microsoft.PowerShell.\*, Microsoft.WSMan.\* | Select-Object -Property Name,@{Name='Parameters';Expression={(Get-Command $_).Parameters.Keys}}
```

The old expression was: `;Expression={$_.Parameters.Keys}}`

```powershell
# run in v2, export all core cmdlets, name and parameters
Get-Command -Module Microsoft.PowerShell.*, Microsoft.WSMan.* | Select-Object -Property Name, @{Name=’Parameters’;Expression={(Get-Command $_).Parameters.Keys}} | Export-Clixml .\v2.xml

# run in v3, export all core cmdlets, name and parameters</span>
Get-Command -Module Microsoft.PowerShell.*, Microsoft.WSMan.* | Select-Object -Property Name ,@{Name=’Parameters’;Expression={(Get-Command $_).Parameters.Keys}} | Export-Clixml .\v3.xml

# run either in v2 or v3 console</span>
$v2 = Import-CliXml .\v2.xml | Sort-Object -Property Name
$v3 = Import-CliXml .\v3.xml | Sort-Object -Property Name

Compare-Object $v2 $v3 -Property Name -IncludeEqual -PassThru | ForEach-Object {
    $Command = $_

    if($_.SideIndicator -eq '==')
    {
        $Command=$_

        $cv2 = $v2 | Where-Object { $_.Name -eq $Command.Name} | Select-Object -ExpandProperty Parameters
        $cv3 = $v3 | Where-Object { $_.Name -eq $Command.Name} | Select-Object -ExpandProperty Parameters

        $compare = Compare-Object $cv2 $cv3

        if ($compare)
        {
            try
            {
                $NewParameters = $compare | Where-Object { $_.SideIndicator -eq '=>' } | ForEach-Object { $_.InputObject + ' (+)'}
                $RemovedParameters = $compare | Where-Object { $_.SideIndicator -eq '=<' } | ForEach-Object { $_.InputObject + ' (-)'}

                "$($command.Name) (!)"
                $NewParameters + $RemovedParameters | Sort-Object | ForEach-Object { "`t$_" }
                "`n"
            }
            catch{}
        }
    }
    elseif ($_.SideIndicator -eq '=>')
    {
        "$($Command.name) (+)`n"
    }
    else
    {
        "($Command.name) (-)`n"

    }
}
```

Result legend: `! = Changed, + = New, and - = Removed`

[1]: http://www.buildwindows.com/
[2]: http://msdn.microsoft.com/en-us/windows/apps/br229516