---
title: Holiday present from PowerShell Magazine – DSC resource module to install and configure Azure Recovery Services Agent
author: Ravikanth C
type: post
date: 2014-12-22T17:03:43+00:00
url: /2014/12/22/holiday-present-from-powershell-magazine-dsc-resource-module-to-install-and-configure-azure-recovery-services-agent/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
Ok, this is not as big as the [PowerShell team&#8217;s holiday gift][1] but something that has been very useful to me. I am sharing the [DSC resource module for installing and configuring Microsoft Azure Recovery Services Agent][2] with the community today. With this, the overall count of [DSC resources in my Github repo goes to 21][3]!

![](/images/ttal-1024x373.png)

All these modules are available in the [PowerShell Gallery][4]. You can [find and install them][5] using the cmdlets in the _PowerShellGet_ module.

<pre class="brush: powershell; title: ; notranslate" title="">Install-Module -Name cMMAgent
Install-Module -Name cWindowsOS
Install-Module -Name cWMIPermanentEvents
Install-Module -Name cMicrosoftAzureRecoveryServices
</pre>

## Introducing cMicrosoftAzureRecoveryServices DSC module

[Microsoft Azure Recovery Services Agent][6] (MARS) is used to connect target systems to Azure Backup or Recovery Services vault. This [DSC resource module][2] intends to build custom resources for installing and configuring MARS Agent on target systems.

<ul class="task-list">
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSAgentInstall">cMARSAgentInstall</a> is used to install Microsoft Azure Recovery Services Agent. This is a composite resource that uses Package resource behind the scenes.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSProxy">cMARSProxy</a> is used to configure the proxy settings for the MARS Agent to connect to the Azure Backup Vault.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSRegistration">cMARSRegistration</a> DSC resource should be used to register a target system with the Azure Backup Vault.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSEncryptionPhrase">cMARSEncryptionPhrase</a> is used to configure the encryption settings for the MARS Agent service.
  </li>
</ul>

At present, this resource module contains four DSC resources that are the fundamental building blocks for using MARS Agent.

#### [][7]{#user-content-using-cmarsagentinstall-resource.anchor}Using cMARSAgentInstall resource

The [cMARSgentInstall][8] is a composite DSC resource. This can be used to install MARS Agent in an unattended manner. Behind the scenes, this resource uses the Package DSC resource.

![](/images/AgentInstall.png)

The _SetupPath_ property is used to specify a local folder path where MARS Agent installer is stored. This can also be the HTTP location for downloading the installer. For example, the following function can be used to get the redirected URL from fwlink on Microsoft site

```
Function Get-TrueURL {
   Param (
      [parameter(Mandatory)]
      $Url
   )
   $req = [System.Net.WebRequest]::Create($url)
   $req.AllowAutoRedirect=$false
   $req.Method="GET"

   $resp=$req.GetResponse()
   if ($resp.StatusCode -eq "Found") {
      return $resp.GetResponseHeader("Location")
   }
   else {
      return $resp.responseURI
   }
}
```

The fwlink to download MARS Agent installer is <https://go.microsoft.com/fwLink/?LinkID=288905&clcid=0x409>.

<pre class="brush: powershell; title: ; notranslate" title="">$TrueUrl = Get-TrueUrl -Url 'https://go.microsoft.com/fwLink/?LinkID=288905&clcid=0x409'
</pre>

Now, this redirected URL can be given as value to the _Path_ property. The Package resource downloads the installer before it attempts to install it on the target system.

Here is a configuration script that uses the cMARSAgentInstall composite resource.

<pre class="brush: powershell; title: ; notranslate" title="">cMARSAgentInstall AgentSetup {
    SetupPath = 'C:\AzureBackup\MARSAgentInstaller.exe'
    EnableWindowsUpdate = $true
    Ensure = 'Present'
}
</pre>

The _EnableWindowsUpdate_ property, when specified, uses Windows Update after the install to check if there are any updates to the agent software. This property is not mandatory.

Once you have installed the MARS Agent, you can register the target system with the Backup Vault. But, before that, if your target system is behind a proxy server, you need to specify the proxy server and proxy port. This can be done using the [cMARSProxy][9] DSC resource.

#### [][10]{#user-content-using-cmarsproxy-resource.anchor}Using cMARSProxy resource

The [cMARSProxy][9] resource can be used to add or remove proxy settings for the Microsoft Azure Recovery Services Agent.

![](/images/Proxy.png)

The only property that is mandatory is the _ProxyServer_. For this, you need to specify a proxy URL (either HTTP or HTTPS). The following configuration script demonstrates this usage.

<pre class="brush: powershell; title: ; notranslate" title="">cMARSProxy MARSProxy {
    ProxyServer = 'https://myProxy'
    Ensure = 'Present'
}
</pre>

DO NOT add the proxy port as a part of the URL. Instead, use the _ProxyPort_ property.

<pre class="brush: powershell; title: ; notranslate" title="">cMARSProxy MARSProxy {
    ProxyServer = 'https://myProxy'
    ProxyPort = 1010
    Ensure = 'Present'
}
</pre>

Optionally, if your proxy server requires authentication, you can specify that using the _ProxyCredential_ property. This is a_PSCredential_ type property and therefore you need to use certificates to encrypt the credentials within the configuration. In case you don&#8217;t have the certificates for your development or test environment, you can pass the plain text credentials.

```
$ProxyCredential = Get-Credential

$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'MARSDemo' }
   )
}

Configuration MARSAgentConfiguration {
   Import-DscResource -ModuleName cMicrosoftAzureRecoveryServices
   Node $AllNodes.NodeName {
      cMARSProxy MARSProxy {
         ProxyServer = 'https://myProxy'
         ProxyPort = 1010
         ProxyCredential = $ProxyCredential
         Ensure = 'Present'
      }
   }
}
```

Once you set the proxy credentials, if you need to change the password alone within the credentials, you can use the_Force_ property to force that change.

```
$ProxyCredential = Get-Credential

$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'MARSDemo' }
   )
}

Configuration MARSAgentConfiguration {
   Import-DscResource -ModuleName cMicrosoftAzureRecoveryServices
   Node $AllNodes.NodeName {
      cMARSProxy MARSProxy {
         ProxyServer = 'https://myProxy'
         ProxyPort = 1010
         ProxyCredential = $ProxyCredential
         Ensure = 'Present'
         Force = $true
      }
   }
}
```

#### Using cMARSRegistration resource

The [cMARSRegistration][12] resource can be used to register the target system with Azure Backup Vault. For registering with the Backup Vault, we need the Vault credentials. These credentials can be downloaded from the [Azure Portal][13] by navigating to the Backup Vault.

![](/images/vaultCred.png)

Once these Vault credentials are downloaded and stored in a local folder, we can use the _cMARSRegistration_ resource configuration to register the server. The _VaultCredential_ property can be used to specify the absolute path to the Vault credentials file.

![](/images/registration.png)

<pre class="brush: powershell; title: ; notranslate" title="">cMARSRegistration AgentRegistration {
    VaultCredential = 'C:\AzureBackup\DSCResourceDemo.VaultCredentials'
    Ensure = 'Present'
}
</pre>

Make a note that the path specified must be absolute path. When the registration is complete, you can see the server listed in the Backup Vault dashboard.

![](/images/Server.png)

Once the target system registration is complete, you can set the encryption pass phrase to encrypt the backup that is going to the Backup Vault. This is done using [cMARSEncryptionPhrase][14] DSC resource.

Note that there is no method to de-register a target system. There is no API available for that. Therefore, when you sent_Ensure=&#8217;Absent&#8217;_, you just see a message that the _Absent_ functionality is not implemented.

You can uninstall the recovery services agent when you don&#8217;t need to backup the system anymore. However, remember that uninstalling the agent does not remove the system registration from the Backup Vault. You need to manually remove the server by navigating to the Backup Vault on the Azure management portal.

#### [][15]{#user-content-using-cmarsencryptionphrase-resource.anchor}Using cMARSEncryptionPhrase resource

The _cMARSEncryptionPhrase_ resource enables you to configure the encryption settings for the MARS Agent Service. This encryption phrase will be used as the encryption key to encrypt the contents of your server backup.

![](/images/encryption.png)

The _EncryptionPassPhrase_ is the only mandatory property and takes plain text string as the passphrase. The resource module internally converts the plain text string to secure string type required for the agent service configuration.

A secure string cannot be directly implemented as the MOF schema does not support a _SecureString_ type. This can be collected as a credential but it does not make a lot of sense. So, I left it as the plain text string for now. This may change in a future release.

<pre class="brush: powershell; title: ; notranslate" title="">cMARSEncryptionPhrase Passphrase {
    EncryptionPassPhrase = 'fawr123456789012345'
    Ensure = 'Present'
}
</pre>

Note that the length of the passphrase must be at least 16 characters.

Once the passphrase is set, if you need to modify the passphrase, you can use the _Force_ property.

<pre class="brush: powershell; title: ; notranslate" title="">cMARSEncryptionPhrase Passphrase {
   EncryptionPassPhrase = 'fawr12345asf2012345'
   Force = $true
   Ensure = 'Present'
}
</pre>

Note that there is no way in the API (AFAIK) to remove the passphrase or encryption settings. Therefore, specifying _Ensure=&#8217;Absent&#8217;_ has no effect on the agent&#8217;s encryption settings.

This is it. I will soon add the functionality to schedule backups to Azure using a DSC resource. Stay tuned.

[1]: /2014/12/18/holiday-gift-from-windows-powershell-team-dsc-resource-kit-wave-9/
[2]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices
[3]: https://github.com/rchaganti/DSCResources
[4]: http://www.powershellgallery.com
[5]: /2014/12/15/exploring-and-installing-dsc-resources-published-in-powershell-gallery/
[6]: http://azure.microsoft.com/en-in/documentation/articles/backup-configure-vault/
[7]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices#using-cmarsagentinstall-resource
[8]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSAgentInstall
[9]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSProxy
[10]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices#using-cmarsproxy-resource
[11]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices#using-cmarsregistration-resource
[12]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSRegistration
[13]: http://manage.windowsazure.com/
[14]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices/DSCResources/cMARSEncryptionPhrase
[15]: https://github.com/rchaganti/DSCResources/tree/master/cMicrosoftAzureRecoveryServices#using-cmarsencryptionphrase-resource