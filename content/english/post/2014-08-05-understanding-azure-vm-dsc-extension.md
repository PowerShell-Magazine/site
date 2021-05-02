---
title: Understanding Azure VM DSC Extension
author: Ravikanth C
type: post
date: 2014-08-05T17:04:31+00:00
url: /2014/08/05/understanding-azure-vm-dsc-extension/
categories:
  - Azure
  - PowerShell DSC
tags:
  - Azure
  - PowerShell DSC

---
The Azure team released the [Azure PowerShell Tools version 0.8.6][1] that includes cmdlets for working with a new Azure VM extension called DSC extension. Using DSC extension, we can push configuration to an Azure VM. In an earlier article, I&#8217;d written about [boot-strapping DSC meta-configuration][2] (LCM) in an Azure VM using the custom script extension. Now, with this new DSC extension, sending DSC configuration (either meta or not) is much more straightforward. In this article, I will explain how to use this new extension.

### Prerequisites

First of all, to be able to use the Azure VM DSC extension, you&#8217;d need [Azure PowerShell Tools version 0.8.6][1]. Second, I assume that you have an existing Azure storage account that can be used to upload the DSC configuration archives. If not, create one and a container within it.

### **Known Issues**

It is good to know these issues upfront before you proceed further.

#1. Do not include -Name parameter with the _Import-DscResource_ in the DSC configuration script. This will result in an error when you try the _Publish-AzureVMDscConfiguration_ cmdlet. I logged [this as an issue][3].

#2. If the Azure VM is configured as a DSC pull client, using _Set-AzureVMDscExtension_ will not enact the configuration. I see no way of force pushing a configuration to a Azure VM that is a pull client. I submitted [this as a feature request][4].

#3. The _Set-AzureVMDscExtension_ cmdlet&#8217;s help content does not tell you that the _Update-AzureVM_ needs to be run. This is a documentation bug.

### Introducing Azure VM DSC extension

When you install the 0.8.6 version of Azure PowerShell Tools, you can see new DSC extension-related cmdlets.

![](/images/dscvm1.png)

There are four cmdlets for this extension.

  * **Get-AzureVMDscExtension** provides a way to get the settings of the DSC extension from the Azure VM.
  * **Set-AzureVMDscExtension** configures the DSC extension in an Azure VM.
  * **Publish-AzureVMDscConfiguration** packs the configuration script and the required modules and uploads the zip archive to the specified Azure storage account.
  * **Remove-AzureVMDscExtension** removes or disables the DSC extension in an Azure VM.

### The Process

Before we can enact DSC configuration in an Azure VM, we need to publish that configuration to an Azure storage container. This can be done using the _Publish-AzureVMDscConfiguration _cmdlet. For the demonstration purposes, I will use the following configuration script:


    Configuration AzureDscDemo {
           Import-DscResource -ModuleName xSystemSecurity
           Node Localhost {
               File DscFile {
                   Type = "Directory"
                   Ensure = "Present"
                   DestinationPath = "C:\Scripts"
               }
               WindowsFeature DscService {
                   Name = "DSC-Service"
                   Ensure = "Present"
               }
    
               xIEESc DscIE {
                   UserRole = "Users"
                   IsEnabled = $false
               }
      	}
    }
If you observe the above configuration script, I am using a custom DSC resource in the node configuration and therefore importing the resource module using the _Import-DscResource_ cmdlet.

The configuration script needs to be saved as a .PS1 file on the local system before we can use the _Publish-AzureVMDscConfiguration_ cmdlet. I saved it as AzureVMConfiguration.ps1.

### Publishing Azure Configuration

The _Publish-AzureVMDscConfiguration_ cmdlet can be used to either generate & upload the configuration archive to the Azure storage account or generate the archive and store it locally. In the first case, we can explicitly specify where the configuration archive should be stored. Otherwise, the cmdlet creates a storage container named _windows-powershell-dsc_ in the storage account you have and upload the zip archive to that account.

<pre class="brush: powershell; title: ; notranslate" title="">Publish-AzureVMDscConfiguration -ConfigurationPath .\AzureVMConfiguration.ps1
</pre>

![](/images/dscvm2.png)

If you have multiple storage accounts, this cmdlet uses the current storage account set either using the _Set-AzureSubscription_ cmdlet or the _Set-AzureStorageAccount_ cmdlet. So, what&#8217;s inside this zip archive? It contains the configuration script that we created as well as any modules imported in the configuration script using the _Import-DscResource_ cmdlet. If you do not want the cmdlet to upload the archive to a storage container, you can use the _-ConfigurationArchivePath_ parameter to specify a local folder where the generated zip archive can be stored.

<pre class="brush: powershell; title: ; notranslate" title="">Publish-AzureVMDscConfiguration -ConfigurationPath .\AzureVMConfiguration.ps1 -ConfigurationArchivePath .\AzureVMConfiguration.ps1.zip
</pre>

![](/images/dscvm3.png)

In my configuration script, I have used _xIEESC_ DSC resource which is a part of the _xSystemSecurity_ module. Therefore, the archive that is generated on my system has the configuration script as well as the _xSystemSecurity_ module folder.

If you want to explicitly specify where the configuration archive must be uploaded, you can use the _-ContainerName_ parameter. The container will be created if it does not already exist.

<pre class="brush: powershell; title: ; notranslate" title="">Publish-AzureVMDscConfiguration -ConfigurationPath .\AzureVMConfiguration.ps1 -ContainerName "dscarchives"
</pre>

And, finally, if you need to upload the configuration archive to a storage account that requires authorization, you can use the -StorageContext parameter to provide the Azure Storage account context.

```
$StorageAccount = 'psdsc'
$StorageKey = 'StorageKey'
$StorageContainer = 'dscarchives'

$StorageContext = New-AzureStorageContext -StorageAccountName $StorageAccount -StorageAccountKey $StorageKey
Publish-AzureVMDscConfiguration -ConfigurationPath .\AzureVMConfiguration.ps1 -ContainerName $StorageContainer -StorageContext $StorageContext
```

When providing the storage context, you must provide the access key for the storage account. Once again, the container will be created if it does not exist.

### Setting Azure VM DSC extension

For enabling and enacting the configuration, we can use the _Set-AzureVMDscExtension_ cmdlet. For using this cmdlet, we need to specify the configuration archive name that is uploaded to the storage account and the name of the configuration command or the identifier used for the configuration command. If you have used a different storage account and container name for the archive upload, you need to specify those details as well. The Azure VM object is specified using the _-VM_ parameter.

```
$vm = Get-AzureVM -ServiceName psdsc -Name CSETest
Set-AzureVMDscExtension -VM $vm -ConfigurationArchive AzureVMConfiguration.ps1.zip -ConfigurationName AzureDscDemo -Verbose -StorageContext $StorageContext -ContainerName $StorageContainer | Update-AzureVM
```


![](/images/dscvm4.png)

At this point, WMF 5.0 July 2014 release gets installed on the Azure VM. This requires a reboot and the Azure VM gets rebooted after the install. Once the VM comes back online, the configuration gets downloaded from the storage account as pending.mof and gets applied.

If your configuration script requires any parameters, you can pass them as a hash table using the _-ConfigurationArgument_ parameter. The _-ConfigurationDataPath_ can be used to specify the path to a .PSD1 file that contains the DSC configuration data. The following code snippet shows how the configuration data needs to be written. This is just a sample and you can refer to [Technet documentation for a detailed example][5].

```
@{
    AllNodes = @(
       @{ NodeName = "localhost"; ScriptPath = "C:\Scripts"; }
    )
}
```


You need to save this as a .PSD1 file and specify the path to it as an argument to _-ConfigurationDataPath_ parameter. This PSD1 file gets uploaded to the storage account to the same container as the configuration script archive. Of course, you need to modify the configuration script to use the configuration data we provide as a PSD1 file. And, whenever you modify the configuration script, it needs to be published again using the _Publish-AzureVMDscConfiguration_ cmdlet.

```
$vm = Get-AzureVM -ServiceName psdsc -Name CSETest
Set-AzureVMDscExtension -VM $vm -ConfigurationArchive AzureVMConfiguration.ps1.zip -ConfigurationName AzureVMConfiguration -Verbose -StorageContext $StorageContext -ContainerName $StorageContainer -Force -ConfigurationDataPath 'ConfigData.psd1' | Update-AzureVM
```


This is it. You can use the _Get-AzureVMDscExtension_ cmdlet to get the extension settings.

<pre class="brush: powershell; title: ; notranslate" title="">$vm = Get-AzureVM -ServiceName psdsc -Name CSETest
Get-AzureVMDscExtension -VM $vm
</pre>
![](/images/5-1024x238.png)

But, how do we access the DSC configuration details from this Azure VM? Simple. We can use CIM sessions and the _Get-DscConfiguration_ cmdlet.

```
$cimsessionoption = New-CimSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck -UseSsl
$cimsession = New-CimSession -Credential (Get-Credential -UserName "AdminUser" -Message "Password") -ComputerName psdsc.cloudapp.net -Port 5986 -SessionOption $cimsessionoption
Get-DscConfiguration -CimSession $cimsession
```


### Removing DSC extension

Finally, we can remove the Azure VM DSC extension using the _Remove-AzureVMDscExtension_ cmdlet.

```
Remove-AzureVMDscExtension -VM $vm -Verbose | Update-AzureVM
```


Note that this command will only disable the Azure VM DSC extension and does not remove the configuration changes done using DSC from the target Azure VM.

In the subsequent articles, we will look at how we can use this extension to bootstrap a new Azure VM.

[1]: /2014/08/05/azure-powershell-tools-version-0-8-6-is-available/
[2]: /2014/07/29/bootstrapping-lcm-meta-configuration-in-a-azure-vm/
[3]: https://github.com/Azure/azure-sdk-tools/issues/2788
[4]: https://github.com/Azure/azure-sdk-tools/issues/2789
[5]: http://technet.microsoft.com/en-us/library/dn249925.aspx

