---
title: DSC composite resource for auto administrator logon
author: Ravikanth C
type: post
date: 2014-10-07T16:00:34+00:00
url: /2014/10/07/dsc-composite-resource-for-auto-administrator-logon/
views:
  - 11224
ratings_users:
  - 1
ratings_score:
  - 5
ratings_average:
  - 5
post_views_count:
  - 1876
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
For my lab setup, I sometimes need to enable auto administrator logon during OS and application deployment. I use DSC for most part of this automation and what is better than a DSC resource for this?

Auto administrator logon is enabled by making changes to the HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\WinLogon registry key. To enable auto administrator logon, we need to add the following values:

  * _AutoAdminLogon_ (REG_SZ) with a value of 1
  * _DefaultUserName_ (REG_SZ) with the user name as the value
  * _DefaultPassword_ (REG_SZ) with the user password as the value
  * _DefaultDomainName_ (REG_SZ) with the domain name of the user as the value

To disable auto administrator logon, we can simply set the _AutoAdminLogon_ to 0 or delete all the above values. Now, coming back to DSC, there is a DSC registry resource. So, there is no need to re-invent the wheel. Instead, we can simply create a composite resource to do this.  The following configuration script combines all the registry entries that need to be configured for the auto administrator logon.

```
Configuration AutoAdminLogon {
   Param (
      [Parameter(Mandatory)]
      [PSCredential] $AutoAdminCredential,

      [Parameter()]
      [ValidateSet("Present","Absent")]
      [String]$Ensure = "Present"
   )

   $Key = 'HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon'

   #Get the default domain name from the credential object
   if ($AutoAdminCredential.GetNetworkCredential().Domain) {
       $DefaultDomainName = $AutoAdminCredential.GetNetworkCredential().Domain
   } else {
       $DefaultDomainName = ""
   }

   Registry DefaultDomainName
   {
      Ensure = $Ensure
      Key = $Key
      ValueName = 'DefaultDomainName'
      ValueData = $DefaultDomainName
   }

   Registry DefaultUserName
   {
      Ensure = $Ensure
      Key = $Key
      ValueName = 'DefaultUserName'
      ValueData = $AutoAdminCredential.GetNetworkCredential().UserName
   }

   Registry DefaultPassword
   {
      Ensure = $Ensure
      Key = $Key
      ValueName = 'DefaultPassword'
      ValueData = $AutoAdminCredential.GetNetworkCredential().Password
   }

   Registry AutoAdminLogon
   {
       Ensure = $Ensure
       Key = $Key
       ValueName = 'AutoAdminLogon'
       ValueData = 1
   }
}
```

In the above configuration, I have used _PSCredential_ type to collect the username and password instead of plain-text strings. Also, the _Ensure_ value indicates whether we want to set the auto administrator logon or remove it. These are the only two parameters we need. The _DefaultDomainName_ will be derived from the _PSCredential_ object. The default _ValueType_ for registry values is REG_SZ and there is no need to provide that property within the resource instance configuration.

To be able to use this configuration as a composite resource, we need to save it as _<ResourceName>.Schema.psm1_. In my example, I saved it as _AutoAdminLogon.Schema.psm1_. Then, we need to create a module manifest for this. This can be done using the _New-ModuleManifest_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">New-ModuleManifest -Path .\AutoAdminLogon.psd1 -RootModule .\AutoAdminLogon.Schema.psm1
</pre>

Once we have the manifest file created, we need to store this in the modules location. Here is how I stored it.

![](/images/22.png)

We need to create a DSCResources folder and store the PSM1 and PSD1 files under that. This can, now, be used to configure _AutoAdminLogon_ using DSC. Here is a sample configuration script:

```
$ConfigData = @{
   AllNodes = @(
      @{ NodeName = "*"; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'WMF5-1' }
   )
}

Configuration Demo {
   Param (
     [pscredential]
     $AutoAdminCredential
   )

   Import-DscResource -Name AutoAdminLogon

   AutoAdminLogon Demo2 {
      AutoAdminCredential = $AutoAdminCredential
      Ensure = "Present"
   }
}

Demo -ConfigurationData $ConfigData -AutoAdminCredential (Get-Credential)
```

This configuration script ensures that auto administrator logon gets set on the target system. I have used plain-text credentials in this example. However, this should work seamlessly with encrypted credentials as well.