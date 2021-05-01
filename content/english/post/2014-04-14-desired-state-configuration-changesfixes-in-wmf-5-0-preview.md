---
title: Desired State Configuration changes/fixes in WMF 5.0 Preview
author: Ravikanth C
type: post
date: 2014-04-14T16:00:28+00:00
url: /2014/04/14/desired-state-configuration-changesfixes-in-wmf-5-0-preview/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
If you were under a rock and just came out, here is a bit of news for you. Microsoft released [Windows Management Framework 5.0][1] preview that includes changes to [DSC][2] and a whole new feature called OneGet. We will talk more about OneGet some other time but I will list a set of changes to DSC I have been able to figure out.

### Get-DscResource is faster

If you have ever used _Get-DscResource_ in WMF 4.0 and you have a whole bunch of community resources to copy to your modules path, you can go and grab a coffee. _Get-DscResource_ was SLOW! This is fixed in WMF 5.0. It just works! 🙂

### Credential property of Archive resource

In the earlier article, I complained that the [Archive resource must have a Credential property][3] to make it easy for accessing UNC path. This is included in WMF 5.0. Now, we can write a simple configuration script to unpack a ZIP archive from a network share on to a target node. Easy! No need to use [File resource][4] as a intermediate step or [assign computer account permissions][5].

    $ConfigurationData = @{
        AllNodes = @(
            @{
                NodeName=Server01
                PSDscAllowPlainTextPassword=$true
             }
        )
    }
    
    Configuration ArchiveDemo {
        param (
            [Parameter(Mandatory=$true)]
            [ValidateNotNullOrEmpty()]
            [String]$Path,
            [Parameter(Mandatory=$true)]
            [PSCredential]$Credential,
    
            [Parameter(Mandatory=$true)]
            [String]$Destination
    	)
    
        Node $AllNodes.NodeName {
            Archive ArchiveDemo {
                Path = $Path
                Destination = $Destination
                Credential = $Credential
            }
        }
    }
    
    ArchiveDemo -ConfigurationData $configurationData -Path "\\Home-desk\Test\Videos.zip" -Credential (Get-Credential) -Destination C:\Videos
As a best practice, you should try and deploy certificates when using Credential property of any DSC resource that supports it. Using the plain text password isn&#8217;t a secure way of transporting credentials.

### Set multiple file or folder attributes

If you have used the _File_ resource, you will know that the _Attributes_ property, although a _String_ array type, cannot really take multiple values. So, you can now set multiple file/folder attributes by using a single resource configuration.

<pre class="brush: powershell; title: ; notranslate" title="">Configuration FileDemo {
    Node Server01 {
        File FileDemo {
            DestinationPath = "C:\Scripts\test.dll"
            Attributes = "System","Hidden"
        }
    }
}
</pre>

### HTTPS URL for Package Path

The Package resource in WMF 4.0 supported only HTTP URL for downloading an online MSI or EXE package for deployment. With the WMF 5.0 preview, the support for HTTPS download URLs is enabled.

```
Configuration PackageDemo {
    Node localhost {
        Package PackageDemo {
            Path = "https://psmag.blob.core.windows.net/downloads/7z920-x64.msi"
            Name = "7-Zip 9.20 (x64 edition)"
            ProductId = "23170F69-40C1-2702-0920-000001000000"
        }
    }
}
PackageDemo
```

Yet another change in Package resource is the pre-validation for the ProductId of the MSI. In WMF 4.0, the ProductId is not validated until the package install is complete. With the WMF 5.0 preview, this is now changed. The new _Get-MsiProductEntry_ in the Package resource module (this is not exported outside the module) is used to get the ProductId from MSI before starting the install. Also, providing an exact Product Name as the value of Name property is now mandatory.

Another change I have noticed but not explored is the _DebugMode_ property in Local Configuration Manager (LCM). I will update this post as I discover more DSC goodness.

[Update] Abhik from the PowerShell team posted a great [overview of LCM DebugMode][6] in WMF 5.0 preview.

[1]: http://www.microsoft.com/en-us/download/details.aspx?id=42316
[2]: /tags/powershell-dsc/
[3]: /2013/09/27/connect-suggestion-dsc-archive-resource-needs-the-credential-attribute/
[4]: /2013/09/26/using-the-credential-attribute-of-dsc-file-resource/
[5]: /2013/09/02/copying-powershell-modules-and-custom-dsc-resources-using-dsc/
[6]: http://blogs.msdn.com/b/powershell/archive/2014/04/22/debug-mode-in-desired-state-configuration.aspx

