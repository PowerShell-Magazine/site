---
title: DSC Resource for Mounting and Dismounting ISO images
author: Ravikanth C
type: post
date: 2014-10-15T16:00:32+00:00
url: /2014/10/15/dsc-resource-for-mounting-and-dismounting-iso-images/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC
---
**Note**: This DSC resource works only on Windows Server 2102 and above or Windows 8 and above. Also, this article assumes that you understanding DSC custom resource development. If not, please head to <http://technet.microsoft.com/en-us/library/dn249927.aspx> for information on building custom DSC resources. The DSC resource module code from this article can be downloaded from my [Github repository][1].

This is yet another resource from my collection of custom DSC resources. As a part of deployment automation, I usually mount application software ISO images and perform install from the mounted location. While this is trivial, the real use case is to ensure that the mounted disk image has the drive letter I specify. Without this, I will have to find ways to retrieve the drive letter assigned to the mounted image. So, the DSC resource that I created helps me assigning the drive letter I specify to the mounted image and therefore making it easy to continue the automation using DSC. Here is the DSC resource module file.

```
Function Get-TargetResource {
	[OutputType([Hashtable])]
	param (
		[Parameter(Mandatory)]
		[string] $ImagePath,

		[Parameter(Mandatory)]
		[string] $DriveLetter,

		[Parameter()]
		[ValidateSet('Present','Absent')]
		[string] $Ensure = 'Present'
	)

    $Configuration = @{
        ImagePath = $ImagePath
        DriveLetter = $DriveLetter
    }

    $DiskImage = Get-DiskImage -ImagePath $ImagePath -ErrorAction Stop
    if ($DiskImage.Attached) {
        if (($DiskImage | Get-Volume).DriveLetter -eq $DriveLtter) {
	        $Configuration.Add('Ensure','Present')
        } else {
    	    $Configuration.Add('Ensure','Absent')
        }
    }

    $Configuration
}

Function Set-TargetResource {
	[CmdletBinding()]
	param (
		[Parameter(Mandatory)]
		[string] $ImagePath,

        [Parameter(Mandatory)]
        [string] $DriveLetter,

        [Parameter()]
        [ValidateSet('Present','Absent')]
        [string] $Ensure = 'Present'
	)

	$DriveLetter = $DriveLetter + ':'

    if ($Ensure -eq 'Present')
    {
    	Write-Verbose "Mounting Disk image"
    	$DiskImage = Mount-DiskImage -ImagePath $ImagePath -NoDriveLetter -PassThru -ErrorAction Stop | Get-Volume
    	$DiskVolume = Get-CimInstance -ClassName Win32_Volume | Where-Object { $_.DeviceID -eq $DiskImage.ObjectId }

    	Write-Verbose "Setting Drive Letter"
    	Set-CimInstance -Property @{DriveLetter= $DriveLetter } -InputObject $DiskVolume -ErrorAction Stop
    }
    else
    {
    	Write-Verbose "Dismounting disk image"
    	Dismount-DiskImage -ImagePath $ImagePath
    }
}

Function Test-TargetResource {
    [CmdletBinding()]
    [OutputType([Boolean])]
    param (
    	[Parameter(Mandatory)]
    	[ValidateScript(
    	{
    		([System.IO.Path]::GetExtension($_) -eq '.iso') -and (Test-Path $_)
    	}
    	)]
    	[string] $ImagePath,

        [Parameter(Mandatory)]
        [ValidateScript({-not (Test-Path $_)})]
        [string] $DriveLetter,

        [Parameter()]
        [ValidateSet('Present','Absent')]
        [string] $Ensure = 'Present'
    )

    $DiskImage = Get-DiskImage -ImagePath $ImagePath -ErrorAction Stop
    if ($DiskImage.Attached)
    {
    	if (($DiskImage | Get-Volume).DriveLetter -eq $DriveLetter)
    	{
    		$MountExists = $true
    		Write-Verbose 'Disk image is mounted with the same drive letter'
    	}
    	else
    	{
    		$MountExists = $false
    		Write-Verbose 'Disk image is mounted but with a different drive letter'
    	}
    }

    if ($MountExists)
    {
    	if ($Ensure -eq 'Present')
    	{
		    Write-Verbose 'disk image is already mounted. No action needed'
    		$true
    	}
    	else
    	{
    		Write-Verbose 'disk image is mounted while it should not'
    		$false
    	}
    }
    else
    {
    	if ($Ensure -eq 'Absent') {
    		Write-Verbose 'disk image is not mounted. No action needed'
    		$true
    	}
    	else
    	{
    		Write-Verbose 'disk image is not mounted while it should'
    		$false
    	}
    }
}

Export-ModuleMember -Function *-TargetResource
```

This is simple. In the _Set-TargetResource_ function, I used _Set-CimInstance_ cmdlet to set the drive letter I need for the mounted image.

```
[ClassVersion("1.0"), FriendlyName("DiskImage")]
class DiskImage : OMI_BaseResource
{
    [Key] string ImagePath;
    [Key] string DriveLetter;
    [write,ValueMap{"Present", "Absent"},Values{"Present", "Absent"}] string Ensure;
};
```


If you observe the above MOF schema definition, I have defined both _ImagePath_ and _DriveLetter_ as _Key_ properties which means a combination of these two identifies a unique configuration. Without this, if a image is already mounted, I still have to go and search for its drive letter. By making this a composite key, if that combination is not configured already, I just go configure it. Simple! If the image is mounted already with a different drive letter, re-mounting it changes the driver letter. Simple, again! 🙂

So, here is how you use this resource

```
Configuration DiskImageDemo {
   Import-DscResource -Module PSMag
   DiskImage ISO {
      ImagePath = 'C:\Images\en_windows_server_2012_r2_with_update_x64_dvd_4065220.iso'
      DriveLetter = 'Z'
      Ensure = "Absent"
   }
}

DiskImageDemo
```

Here is the output from one such configuration enact process.

![](/images/dscin2.png)

As you know, _Mount-DiskImage_ can be used to mount VHDX and VHD files too. So, this resource can be easily updated to add that support as well. Watch this space.

You can download this DSC resource from my [Github repository][1].

[1]: https://github.com/rchaganti/DSCResources/tree/master/DSCResources/DiskImage

