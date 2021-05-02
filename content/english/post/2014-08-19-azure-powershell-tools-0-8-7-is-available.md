---
title: Azure PowerShell Tools 0.8.7 is available
author: Ravikanth C
type: post
date: 2014-08-19T06:14:09+00:00
url: /2014/08/19/azure-powershell-tools-0-8-7-is-available/
categories:
  - Azure
  - News
tags:
  - Azure
  - News

---
Just a few hours ago, [Azure PowerShell Tools version 0.8.7](https://github.com/Azure/azure-sdk-tools/releases) was released. This release includes enhancements to the existing cmdlets.

Here is what’s updated in this release:

- Updated Newtonsoft.Json dependency to 6.0.4
- Compute
  - Windows Azure Diagnostics (WAD) Version 1.2: extension cmdlets for Iaas And PaaS
    - Set-AzureVMDiagnosticsExtension
    - Get-AzureVMDiagnosticsExtension
    - Set-AzureServiceDiagnosticsExtension
    - Get-AzureServiceDiagnosticsExtension
  - Get-AzureDeployment: added CreatedTime and LastModifiedTime to output
  - Get-AzureVM: added Hostname property
  - Implemented CustomData support for Azure VMs
- Websites
  - Added RoutingRules parameter to Set-AzureWebsite to expose Testing in Production (TiP) and returned from Get-AzureWebsite
  - Get-AzureWebsiteMetric to return web site metrics
  - Get-AzureWebHostingPlan
  - Get-AzureWebHostingPlanMetric to return metrics for the servers in the web hosting plan
- SQL Database
  - Get-AzureSqlRecoverableDatabase parameter simplification and return type changes
  - Set-AzureSqlDatabaseRecovery parameter and return type changes
- HDInsight
  - Added support for provisioning of HBase clusters into Virtual Networks.

I will write about the VM diagnostics extension and other enhancements in an upcoming post.

Meanwhile, you can use the following script to get the most recent MSI downloaded from the Azure PowerShell Tools release page:

```
Function Test-Url {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [String] $Url
    )

Process {
    if ([system.uri]::IsWellFormedUriString($Url,[System.UriKind]::Absolute)) {
        $true
    } else {
        $false
    }
}

}

Function Get-AzurePowerShellMSI {
    [CmdletBinding()]
    param (
        [String]$Url = 'https://github.com/Azure/azure-sdk-tools/releases',
        [Switch]$DownloadLatest,
        [String]$DownloadPath = "$env:USERPROFILE\Downloads",
        [Switch]$Passthru
    )

Process {
    $Msi = @()
    $doc = Invoke-WebRequest -Uri $Url

foreach ($href in ($doc.links.href -ne '')) {
    if ((Test-Url -Url $href) -and $href.EndsWith('.msi')) {
        $Msi += New-Object -TypeName PSObject -Property @{
            "Url" = $href
            "Version" = ([Regex]::Match($href,'\bv?[0-9]+\.[0-9]+\.[0-9]+(?:\.[0-9]+)?\b')).Value
        }
    }
}

if ($DownloadLatest) {
    if (-not (Test-Path $DownloadPath -PathType Container)) {
        Write-Warning "${DownloadPath} does not exist. Creating it ..."
        New-Item -Path $DownloadPath -ItemType Directory -ErrorAction Stop | Out-Null
    }

$MsiToInstall = $msi | Sort -Property Version -Descending | Select -First 1
$MsiFullPath = "${DownloadPath}\$(Split-Path -Path $MsiToInstall.Url -Leaf)"

if (-not (Test-Path -Path $MsiFullPath)) {
    Write-Verbose "Starting MSI download from $($MsiToInstall.Url)"
    Start-BitsTransfer -Source $MsiToInstall.Url -Destination $MsiFullPath -ErrorAction Stop
} else {
    Write-Warning ("{0} already exists at {1}. No action needed." -f $MsiFileName, $DownloadPath)
}

if ($Passthru) {
    return $MsiFullPath
}

} else {
    $Msi
}

}

}
```

This is a simple function and can be used in the following manner:

```
#get a list of all MSIs
Get-AzurePowerShellMSI -Passthru

#Download the latest MSI
Get-AzurePowerShellMSI -DownloadLatest -Verbose -Passthru

#Download the latest MSI to a specific destination
Get-AzurePowerShellMSI -DownloadLatest -DownloadPath C:\MyDownloads1 -Verbose -Passthru
```

[1]: https://github.com/Azure/azure-sdk-tools/releases