---
title: '#PSTip Restarting PowerShell as an administrator'
author: Shay Levy
type: post
date: 2014-03-04T19:00:43+00:00
url: /2014/03/04/pstip-restarting-powershell-as-an-administrator/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 2.0 or above.

I bet this sounds familiar to you. You start working in the PowerShell console or in the ISE, and then at one point you figure out that a command you&#8217;re executing fails because your session is not elevated! You start saving your work so you can close the session and reopen it &#8220;as an administrator&#8221;.

Situations like this can be very irritating ( to me anyway), and that&#8217;s why I wrote _Restart-Host_ function.


    function Restart-Host
    {
        [CmdletBinding(SupportsShouldProcess,ConfirmImpact='High')]
        Param(
            [switch]$AsAdministrator,
            [switch]$Force
        )
    
        $proc = Get-Process -Id $PID
        $cmdArgs = [Environment]::GetCommandLineArgs() | Select-Object -Skip 1
    
        $params = @{ FilePath = $proc.Path }
        if ($AsAdministrator) { $params.Verb = 'runas' }
        if ($cmdArgs) { $params.ArgumentList = $cmdArgs }
    
        if ($Force -or $PSCmdlet.ShouldProcess($proc.Name,"Restart the console"))
        {
            if ($host.Name -eq 'Windows PowerShell ISE Host' -and $psISE.PowerShellTabs.Files.IsSaved -contains $false)
            {
                if ($Force -or $PSCmdlet.ShouldProcess('Unsaved work detected?','Unsaved work detected. Save changes?','Confirm'))
                {
                    foreach ($IseTab in $psISE.PowerShellTabs)
                    {
                        $IseTab.Files | ForEach-Object {
    
                            if ($_.IsUntitled -and !$_.IsSaved)
                            {
                                $_.SaveAs($_.FullPath,[System.Text.Encoding]::UTF8)
                            }
                            elseif(!$_.IsSaved)
                            {
                                $_.Save()
                            }
                        }
                    }
                }
                else
                {
                    foreach ($IseTab in $psISE.PowerShellTabs)
                    {
                        $unsavedFiles = $IseTab.Files | Where-Object IsSaved -eq $false
                        $unsavedFiles | ForEach-Object {$IseTab.Files.Remove($_,$true)}
                    }
                }
            }
    
            Start-Process @params
            $proc.CloseMainWindow()
    	}
    }
When invoked from powershell.exe, _Restart-Host_ will close your current session and reopen it as an administrator while restoring the parameters used to load your current session.

When invoked from the ISE you will also get the chance to save any unsaved scripts.