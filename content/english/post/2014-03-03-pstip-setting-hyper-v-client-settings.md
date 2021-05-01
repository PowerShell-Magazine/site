---
title: '#PSTip Setting Hyper-V client settings'
author: Shay Levy
type: post
date: 2014-03-03T19:00:33+00:00
url: /2014/03/03/pstip-setting-hyper-v-client-settings/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

When working with Hyper-V in Windows Server 2012 or Windows 8 and higher, you can set various Hyper-V host settings via the Hyper-V Settings dialog. Every option under the _&#8220;Server&#8221;_ section is settable via the [_Set-VMHost_][1] cmdlet.

![](/images/HVServerSettings.png)

The dialog also includes a _&#8220;User&#8221;_ section. With this section you can control where keyboard key combination will run, which key combination will be used to release the mouse key (getting out of the VM window bounds), and more.

![](/images/HVClientSettings1.png)

However, User settings cannot be set using any of the Hyper-V cmdlets. They are a part of the Hyper-V Manager MMC snap-in. The Hyper-V Manager keeps tracking of its settings via a series of config files:

```
PS> $path = "$env:APPDATA\Microsoft\Windows\Hyper-V\Client\1.0"
PS> Get-ChildItem $path

	Directory: C:\Users\shay\AppData\Roaming\Microsoft\Windows\Hyper-V\Client\1.0

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---         2/15/2014  12:22 PM        902 clientsettings.config
-a---         2/11/2014  10:06 PM        759 user.config
-a---         2/11/2014  10:06 PM       1511 virtmgmt.VMBrowser.config
-a---          2/5/2014  10:41 AM        622 vmconnect.config
-a---         12/2/2013  10:02 AM       1404 vmwizards.config
```

The settings in the _&#8220;User&#8221;_ section in Hyper-V Manager are stored in the _clientsettings.config_ file. You can take a peek of the content with the _Get-HVClientSettings_ function. Note that the file might not exist if you haven&#8217;t made a change to the _&#8220;User&#8221;_ section in Hyper-V Manager. The values you get back are also version dependant. Prior to Server 2012 R2/Windows 8.1, you will only see the first two values. Additionally, there seems to be one setting that is missing from the UI, _&#8216;UseAllMonitors&#8217;, _which I guess is probably related to the Remote Desktop Multimon feature available in Remote Desktop Client 7.0.


    function Get-HVClientSettings
    {
        $path = "$env:APPDATA\Microsoft\Windows\Hyper-V\Client\1.0\clientsettings.config"
        if(Test-Path $path)
        {
            $ClientSettings = Get-Content $path
            $settings = $ClientSettings.configuration.'Microsoft.Virtualization.Client.ClientVirtualizationSettings'
            $settings.setting
        }
        else
        {
            Write-Error "'$path' was not found"
        }
    }
    
    PS> Get-HVClientSettings | Select-Object Name,Value
    name                     value
    ----                     -----
    VMConnectKeyboardOption  Remote
    VMConnectReleaseKey      Shift
    VMConnectUseEnhancedMode True
    DesktopSize              1366, 768
    UseAllMonitors           True
As already stated, none of the above can be changed using the _Hyper-V_ module. But now that you know where the settings reside&#8230;

Here&#8217;s a function that will help you automate the update of User settings.


    function Set-HVClientSettings
    {
        [CmdletBinding()]
        param(
            [ValidateSet('Remote','Local','FullScreen')]
            [string]$ConnectKeyboardOption,
    
            [ValidateSet('LeftArrow','RightArrow','Space','Shift')]
            [string]$ReleaseKey,
    
            [bool]$UseEnhancedMode,
    
            [bool]$UseAllMonitors,
    
            [switch]$RestartClient
        )
    
        $requiredVersion = $PSVersionTable.BuildVersion -lt '6.3.9600'
        $path = "$env:APPDATA\Microsoft\Windows\Hyper-V\Client\1.0\clientsettings.config"
    
        if(Test-Path $path)
        {
            $ClientSettings = Get-Content $path
            $settings = $ClientSettings.configuration.'Microsoft.Virtualization.Client.ClientVirtualizationSettings'
    
            if($ConnectKeyboardOption)
            {
                $settings.SelectSingleNode("//setting[@name='VMConnectKeyboardOption']").value = $ConnectKeyboardOption
            }
    
            if($PSCmdlet.MyInvocation.BoundParameters.ContainsKey('UseEnhancedMode'))
            {
                if($requiredVersion)
                {
                    Write-Warning "'UseEnhancedMode' is available only in Windows 8.1/Windows server 2012 R2 and higher."
                }
                else
                {
                    $settings.SelectSingleNode("//setting[@name='VMConnectUseEnhancedMode']").value = "$UseEnhancedMode"
                }
            }
    
            if($PSCmdlet.MyInvocation.BoundParameters.ContainsKey('UseAllMonitors'))
            {
                if($requiredVersion)
                {
                    Write-Warning "'UseAllMonitors' is available only in Windows 8.1/Windows server 2012 R2 and higher."
                }
                else
                {
                    $settings.SelectSingleNode("//setting[@name='UseAllMonitors']").value = "$UseAllMonitors"
                }
            }
    
            if($ReleaseKey)
            {
                $settings.SelectSingleNode("//setting[@name='VMConnectReleaseKey']").value = $ReleaseKey -replace 'arrow'
            }
    
            $ClientSettings.Save($path)
    
            if($RestartClient)
            {
                Get-Process | Where-Object MainWindowTitle -eq 'Hyper-V Manager' | Stop-Process -Force
    
                if(Test-Path "$env:ProgramFiles\Hyper-V\virtmgmt.msc")
                {
                    # 2008 R2 location
                    & "$env:ProgramFiles\Hyper-V\virtmgmt.msc"
                }
                else
                {
                    virtmgmt.msc
                }
            }
        }
        else
        {
            Write-Error "'$path' was not found"
        }
    }
The function also supports the option to restart Hyper-V Manager as changes will only apply to new instances of virtual machine connections.

[1]: http://technet.microsoft.com/en-us/library/hh848524.aspx