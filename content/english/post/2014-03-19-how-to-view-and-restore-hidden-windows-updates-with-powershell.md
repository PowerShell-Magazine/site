---
title: How to view and restore hidden Windows Updates with PowerShell
author: Emin Atac
type: post
date: 2014-03-19T18:00:44+00:00
url: /2014/03/19/how-to-view-and-restore-hidden-windows-updates-with-powershell/
categories:
  - How To
tags:
  - How To

---
<span style="line-height: 1.5em;">When you visit Windows Update, you can hide an update to avoid being prompted to install it again at next scan. If </span>you&#8217;ve<span style="line-height: 1.5em;"> ever hidden an update, </span>you&#8217;ve<span style="line-height: 1.5em;"> also the ability to restore hidden updates in the Windows Update control panel applet by clicking the </span><em style="line-height: 1.5em;">“restore hidden updates”</em> <span style="line-height: 1.5em;">button. </span>However, over time, you’ll forget what update(s) you&#8217;ve hidden. There’s no built-in way to show hidden updates.

![](/images/image0001.jpg)

Let’s see how PowerShell can help in this situation and define 2 functions and a filter to view already hidden updates.


    Function Get-WindowsUpdate {
        [Cmdletbinding()]
        Param()
    
        Process {
            try {
                Write-Verbose "Getting Windows Update"
                $Session = New-Object -ComObject Microsoft.Update.Session            
                $Searcher = $Session.CreateUpdateSearcher()            
                $Criteria = "IsInstalled=0 and DeploymentAction='Installation' or IsPresent=1 and DeploymentAction='Uninstallation' or IsInstalled=1 and DeploymentAction='Installation' and RebootRequired=1 or IsInstalled=0 and DeploymentAction='Uninstallation' and RebootRequired=1"            
                $SearchResult = $Searcher.Search($Criteria)           
                $SearchResult.Updates
            } catch {
                Write-Warning -Message "Failed to query Windows Update because $($_.Exception.Message)"
            }
        }
    }
    
    Function Show-WindowsUpdate {
        Get-WindowsUpdate |
        Select Title,isHidden,
            @{l='Size (MB)';e={'{0:N2}' -f ($_.MaxDownloadSize/1MB)}},
            @{l='Published';e={$_.LastDeploymentChangeTime}} |
        Sort -Property Published
    }
Now to show all hidden updates, you only need to do the following:

<pre class="brush: powershell; title: ; notranslate" title="">Show-WindowsUpdate | Where { $_.isHidden }| Out-GridView
</pre>

![](/images/image00021.jpg)

To be able to restore all hidden updates, an additional advanced function is required.


    Function Set-WindowsHiddenUpdate {
        [Cmdletbinding()]
    
        Param(
            [Parameter(ValueFromPipeline=$true,Mandatory=$true)]
            [System.__ComObject[]]$Update,
    
            [Parameter(Mandatory=$true)]
            [boolean]$Hide
        )
    
        Process {
            $Update | ForEach-Object -Process {
                if (($_.pstypenames)[0] -eq 'System.__ComObject#{c1c2f21a-d2f4-4902-b5c6-8a081c19a890}') {
                    try {
                        $_.isHidden = $Hide
                        Write-Verbose -Message "Dealing with update $($_.Title)"
                    } catch {
                        Write-Warning -Message "Failed to perform action because $($_.Exception.Message)"
                    }
                } else {
                    Write-Warning -Message "Ignoring object submitted"
                }
            }
        }
    }
Let’s first hide four random updates:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WindowsUpdate | Get-Random -Count 4 |
Set-WindowsHiddenUpdate -Hide $true -Verbose
</pre>

To restore all hidden updates, we can simply do:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WindowsUpdate| Set-WindowsHiddenUpdate -Hide $false -Verbose
</pre>

But if we want to restore only some of them, that’s also possible:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WindowsUpdate | Where { $_.isHidden } | Out-GridView -PassThru |
Set-WindowsHiddenUpdate -Hide $false -Verbose
</pre>

In Europe, Windows Update proposes an update called on my Windows 8.1 the Microsoft Browser Choice Screen Update for EEA Users of Windows 8.1 for x64-based Systems (KB976002)

The problem with this update is that it’s marked as a permanent component and thus cannot be uninstalled.

With the above functions, we are now able to hide this update like this:

<pre class="brush: powershell; title: ; notranslate" title="">Get-WindowsUpdate |
Where { $_.Title -match 'Microsoft Browser Choice Screen Update'} |
Set-WindowsHiddenUpdate -Hide $true -Verbose
</pre>
![](/images/image0031.jpg)