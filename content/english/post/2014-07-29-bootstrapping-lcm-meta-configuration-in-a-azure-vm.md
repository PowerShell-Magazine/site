---
title: Bootstrapping LCM meta-configuration in a Azure VM
author: Ravikanth C
type: post
date: 2014-07-29T16:53:15+00:00
url: /2014/07/29/bootstrapping-lcm-meta-configuration-in-a-azure-vm/
categories:
  - Azure
  - PowerShell DSC
tags:
  - Azure
  - PowerShell DSC

---
I have been a very active user of DSC both in the lab and on all my Azure VMs. One of the things I always do is to configure my Azure VMs to get configuration from a pull service hosted in my Azure subscription. I want these Azure VMs to have the custom meta-configuration as soon as they get created. The custom script extension for the Azure VMs can be used to do this. However, this is not as straightforward as using the _Set-DscLocalConfigurationManager_ cmdlet. In this article, I will explain the method to achieve this and explain each step in this process. You can use these steps to bootstrap configuration other than meta-configuration too.

In an earlier article, I wrote about [custom script extension][1] in Azure. The custom script extension lets us run PowerShell scripts in a Azure VM. If you have used the Azure Management portal recently to create a VM from the gallery, you must have noticed the change in the VM configuration page.

![](/images/AzureVMConfig1.png)

We can select a script (ps1) file from either the local storage or Azure storage and run it after the VM provisioning is complete. We can specify arguments to the script using the arguments textbox shown in the VM configuration screen. I use this method to bootstrap my Azure VMs with the LCM configuration required for my deployment. This method can be used to trigger configuration pull as well. We will see that towards the end of this article.

### Automating LCM meta-configuration

Let us start by looking at the script that generates and applies the meta-configuration. We will understand how this works before we proceed to create an Azure VM that uses this script.

```
param (
   [guid]$ConfigurationId = [guid]::NewGuid().Guid,
   [string]$PullServerUrl
)

Configuration LCMConfig {
   Node $env:COMPUTERNAME {
       LocalConfigurationManager {
           ConfigurationID = "$ConfigurationId"
           RefreshMode = "Pull"
           DownloadManagerName = "WebDownloadManager"
           DownloadManagerCustomData = @{ServerUrl=$PullServerUrl;AllowUnSecureConnection='True'}
       }
   }
}

$mof = LCMConfig

#Create a CIM Session to the localhost
#Need to skip CA and CN checks
$CimSessionOption = New-CimSessionOption -SkipCNCheck -SkipCACheck -UseSsl
$CimSession = New-CimSession -ComputerName $env:COMPUTERNAME -Port 5986 -SessionOption $CimSessionOption 

#Invoke Set-DscLocalConfigurationManager
Set-DscLocalConfigurationManager -CimSession $CimSession -Path $mof.Directory.FullName
```

I have copied this script to one of my Azure storage containers  as LCM.ps1. Let us dissect the script now.

We have a _Configuration_ script block that is used to specify the LCM meta-configuration. Through this, we are configuring the LCM as a pull client to a REST-based pull service. This pull service is also hosted on Azure and is a HTTP endpoint. So, I am setting the _AllowUnSecureConnection_ to _True_. The values for the _ConfigurationId_ and _ServerUrl_ within the _DownloadManagerCustomData_ are passed as the script arguments. Once we have the _Configuration_ command, we run the same and store the MOF file object in _$mof_.

In a normal scenario, we can use _Set-DscLocalConfigurationManager_ cmdlet with just the _-Path_ parameter to apply the meta-configuration in the generated MOF file. However, this won&#8217;t work in a Azure VM for two reasons:

  1. **The default WinRM endpoint is HTTPS-based.** So, we need to create a CIM session and use the SSL endpoint and 5896 port number. Although it is possible to provision a HTTP-based WinRM endpoint using Azure PowerShell cmdlets, I prefer using the HTTPS endpoints in the VM.
  2. **The certificate deployed for WinRM endpoint uses the cloud service name as the subject name**. So, even if we use CIM session with SSL, the enact process would fail because the local host name would not match the certificate&#8217;s subject name.

This is what we solved in the script by creating CIM session options and a CIM session. The _-SkipCNCheck_ and _-SkipCACheck_ parameters of the _New-CimSessionOption_ cmdlet ensure that the certificate&#8217;s subject name gets ignored. We, then, create a CIM session that uses the CIM session options. Finally, we use the _-CimSession_ parameter of the _Set-DscLocalConfigurationManager_ cmdlet to enact this meta-configuration.

### Bootstrapping LCM meta-configuration

Now that we understand the method for automating LCM meta-configuration, let us see how we can boot strap this configuration in an Azure VM as it gets created. First of all, make sure you copy the above script to one of the containers in your Azure storage account. In the following script, we will use the Azure custom script extension to execute the LCM configuration script in the Azure VM.

```
$SubscriptionName = "MyCloud"
Select-AzureSubscription -SubscriptionName $SubscriptionName

$ServiceName = "psdsc"
$Location = "Southeast Asia"
$VMName = "CSETest"
$StorageAccountName = "psdsc"
$StorageContainerName = "scripts"

#Create a new cloud service
Set-AzureSubscription -SubscriptionName $SubscriptionName -CurrentStorageAccountName $StorageAccountName
New-AzureService -ServiceName $ServiceName -Location $Location

#Get the OS image reference
$ImageName = (Get-AzureVMImage | Where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | Select-Object -First 1).ImageName

#Create VM Config
$vmConfig = New-AzureVMConfig -Name $VMName -ImageName $ImageName -InstanceSize Small

#Create Provisioning Configuration
$vmProvisioningConfig = Add-AzureProvisioningConfig -VM $vmConfig -Windows -AdminUsername "AdminUser" -Password "Password"

#Set the Azure Script Extension to run the LCM meta-configuration
$vmAzureExtension = Set-AzureVMCustomScriptExtension -FileName LCM.ps1 -VM $vmProvisioningConfig -ContainerName $StorageContainerName -StorageAccountName $StorageAccountName -Argument "-PullServerUrl http://dscdemo.cloudapp.net:8080/PullSvc/DscPullService.svc"

#Create a VM
New-AzureVM -ServiceName $ServiceName -VMs $vmAzureExtension
```

While creating the VM using above script, make sure you use the same location for all the components in the deployment. This means the cloud service, storage account, and the virtual machine should all be in the same location. The Set-AzureVMCustomScriptExtension cmdlet is used to bootstrap the LCM meta-configuration script. I am using the _-Argument_ parameter of this cmdlet to pass the pull service URL. The _ConfigurationId_ for the target system LCM gets generated in the LCM script and there is no need to explicitly it pass it.

This is it. Once the Azure VM gets created, the LCM meta-configuration script gets executed and changes the LCM settings.

### Bonus Tip

When a target system is configured as a pull client, two scheduled tasks get created. One of those tasks, named Consistency, runs every 15 minutes, by default, to check the pull server for any new or updated configuration. So, when bootstrapping pull client LCM settings, if you want to perform an immediate configuration pull, you can trigger the Consistency schedule task using the scheduled task cmdlets. The following example demonstrates this.

```
Get-ScheduledTask -TaskName Consistency | Start-ScheduledTask
```

So, you can add this line at the end of the LCM script and pull the configuration from a pull server immediately after the LCM meta-configuration change.

[1]: /2014/04/30/understanding-azure-custom-script-extension/