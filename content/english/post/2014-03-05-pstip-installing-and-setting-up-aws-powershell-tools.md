---
title: '#PSTip Installing and setting up AWS PowerShell Tools'
author: Ravikanth C
type: post
date: 2014-03-05T19:00:44+00:00
url: /2014/03/05/pstip-installing-and-setting-up-aws-powershell-tools/
categories:
  - AWS
  - Tips and Tricks
tags:
  - AWS
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

I have been working with Amazon Web Services (AWS) and for me the default management tool is their [PowerShell module][1]. I have multiple computers that I use to work with AWS and I rebuild them quite often. So, I needed a method to speed up this process.

What&#8217;s better than a reusable PowerShell function?


    function Install-AWSPowerShellTool {
        [CmdletBinding()]
    
        param (
            [Parameter()]
            [String]
            $Url = "http://sdk-for-net.amazonwebservices.com/latest/AWSToolsAndSDKForNet.msi",
    
            [Parameter()]
            [Switch]
            $UpdateProfile
        )
    
        if (-not ($PSVersionTable.PSVersion -ge [Version]"2.0.0.0")) {
            Throw "PowerShell version must be 2.0 or above"
        }
    
        if (!(New-Object Security.Principal.WindowsPrincipal ([Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)) {
            Throw "This function must be run as an administrator"
        }
    
        Write-Verbose "Downloading AWS PowerShell Tools from ${url}"
        Start-BitsTransfer -Source $url -Destination $env:TEMP -Description "AWS PowerShell Tools" -DisplayName "AWS PowerShell Tools"
    
        Write-Verbose "Starting AWS PowerShell Tools install using ${env:Temp}\$(Split-Path -Path $Url -Leaf)"
        $Process = Start-Process -FilePath "msiexec.exe" -ArgumentList "/i ${env:Temp}\$(Split-Path -Path $Url -Leaf) /qf /passive" -Wait -PassThru
    
        if ($Process.ExitCode -ne 0) {
            Throw "Install failed with exit code $($Process.ExitCode)"
        } else {
            if ($UpdateProfile) {
                if (-not (test-Path (Split-Path $PROFILE))) {
                    Write-Verbose "Creating WindowsPowerShell folder for copying the profile"
                    New-Item -Path (Split-Path $PROFILE) -ItemType Directory -Force | Out-Null
                }
                Write-Verbose "Updating PowerShell Profile at ${PROFILE}"
                Add-Content -Path $PROFILE -Value 'Import-Module -Name "C:\Program Files (x86)\AWS Tools\PowerShell\AWSPowerShell\AWSPowerShell.psd1"' -Force
            }
        }
    }
[1]: http://docs.aws.amazon.com/powershell/latest/reference/Index.html