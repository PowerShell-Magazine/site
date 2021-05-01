---
title: Set keyboard layouts available on logon screen in Windows 8.1
author: Jakub Jareš
type: post
date: 2014-03-24T16:00:02+00:00
url: /2014/03/24/set-keyboard-layouts-available-on-logon-screen-in-windows-8-1/
categories:
  - How To
tags:
  - How To

---
While we worked on Windows 8.1 deployment, we were required to make multiple keyboard layouts available on the Windows logon screen. All our computers came pre-staged with standardized corporate image, so we were not able to put these settings directly into the image. The only possible solution seemed to be using the _Control panel > Region and Language Settings > Administrative > Copy settings&#8230;_

![](/images/image0013.png)

But that would require clicking through the GUI on every staged computer and I needed a way to automate it. I used _ProcessMonitor.exe_ to see what is happening when I click the OK button in the dialog, but unfortunately that did not help.

I needed a different idea, so I started thinking: The logon screen is in fact _LogonUI.exe_ that is running in the context of the System account. I searched the web and found that the settings for normal user are located in: _HKEY\_CURRENT\_USER\Keyboard Layout\Preload_. And mine looked like this:

![](/images/image0021.png)

Current user hive of the _SYSTEM _account is _HKEY_USERS&#46;DEFAULT and luckily Keyboard Layout\Preload key_ was there. I copied my settings there and restarted the station, hoping for the best.

When the station booted back to the logon screen, the correct keyboard layouts were available there, so this approach should work. Copying the Registry values from one place to another might work but I wanted something more intuitive and flexible.

The International module, introduced in Windows 8 and Windows Server 2012, provides _*-WinUserLanguageList_ cmdlets to manage the keyboard layouts of the current user and I wanted my function to integrate with them nicely.

![](/images/image0034.jpg)

The _Set-WinUserLanguageList_ prompts for confirmation and also warns you if some of the provided languages were invalid and I wanted that behavior too, so this is the function I ended up with:

	function Set-WinLogonLanguageList
	{
		[CmdletBinding(ConfirmImpact='High',SupportsShouldProcess=$true)]
	    <#
	    .SYNOPSIS
	    Sets the keyboards visible on the logon screen
	
	    .DESCRIPTION
	    This function provides automated way to copy the keyboard settings of the current user to the Windows logon screen.
	    It provides part of the functionality available in Region and Language Settings control panel.
	    Region and Language Settings &gt; Administrative &gt; Copy settings... ->
	    Copy to New Users and Welcome Screen > Welcome screen and system accounts
	    Computer restart is needed after the change.
	
	    .PARAMETER LanguageList
	    Accepts list of user language objects.
	
	    .PARAMETER Force
	    Forces the change and computer restart without prompting for confirmation.
	
	    .PARAMETER Restart
	    Enables restarting the computer when the settings are updated.
	
	    .EXAMPLE
	    Get-WinUserLanguageList | Set-WinLogonKeyboardList -Force
	
	    Sets the keyboard layouts of the current user to be available on the logon screen without asking for confirmation.
	#>
	
	    param(
	        [Parameter(
	            Mandatory=$true,
	            ValueFromPipeline=$true,
	            ValueFromPipelineByPropertyName = $true
	        )]
	        [Microsoft.InternationalSettings.Commands.WinUserLanguage[]]
	        $LanguageList,
	
	        [Switch]$Force,
	        [Switch]$Restart
	    )
	
	    begin {
	        $list = @()
	    }
	
	    process {
	        foreach ($language in $LanguageList)
	        {
	            $list += $language
	        }
	    }
	
	    end {
	
	        if ($Force -or $PSCmdlet.ShouldProcess(
	            "Windows logon screen",
	            "Set avalilable language(s) to: $($finalList.EnglishName -join ', ')"
	            )
	        ) {
	            $path = "Microsoft.PowerShell.Core\Registry::HKEY_USERS\.Default\Keyboard Layout\preload"
	
	            #remove the current registry settings
	            $current = (Get-Item $path).Property
	            Remove-ItemProperty -Path $path -Name $current -Force
	
	            #remove languages that are not installed on the system
	            if ($list | where { -not $_.Autonym } )
	            {
	                Write-Warning "The list you attempted to set contained invalid langauges which were ignored"
	                $finalList = $list | where Autonym
	            }
	            else
	            {
	                $finalList = $list
	            }
	
	            $languageCode = $finalList.InputMethodTips -replace  ".*:"
	
	            for ($i = 0; $i -lt $languageCode.count; ++$i)
	            {
	                New-ItemProperty -Path $path -Name ($i+1) -Value $languageCode[$i] -PropertyType String -Force | Out-Null
	            }
	
	            if ($languageCode) {
	                #restart only if changes were made
	                if ($Restart -or $Force -or $PSCmdlet.ShouldProcess("Computer","Restart computer to finish the process."))
	                {
	                    Restart-Computer  -Force
	                }
	            }
	        }
	    }
	}
The function requires Windows 8.1 (or Windows 8) and the elevated privileges to run successfully.

At this point one last catch reminded. During the post-deployment tasks the script is executed under the _SYSTEM_ account. So getting the list of languages using the _Get-WinUserLanguageList_ would only re-apply the settings that were already in place. I had to build the list myself and at first I used the following code:

```
$list = New-WinUserLanguageList -Language cs-CZ
$list.Add("en-US")
Set-WinLogonLanguageList -LanguageList $list -Force
```


But then I realized I was making it more complicated than it had to be and used just:

```
Set-WinLogonLanguageList -LanguageList cs-CZ, en-US -Force
```