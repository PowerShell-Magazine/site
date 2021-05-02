---
title: Bootstrapping DSC configuration using Azure VM DSC extension
author: Ravikanth C
type: post
date: 2014-08-07T16:00:46+00:00
url: /2014/08/07/bootstrapping-dsc-configuration-using-azure-vm-dsc-extension/
categories:
  - Azure
  - PowerShell DSC
tags:
  - Azure
  - PowerShell DSC

---
In one of the earlier articles, I demonstrated how we can use the [Azure VM custom script extension to bootstrap DSC LCM meta-configuration][1] after creating a new VM. In that process, we upload a .PS1 file containing the configuration to a storage container and then use the custom script extension to execute that configuration. Within the configuration script, we had to work around some of the issues pertaining to certificate subject name (when using WinRM HTTPS endpoint in Azure VM) and so on using CIM sessions. While all this is trivial, the biggest disadvantage is that there is no explicit support for finding custom DSC resources required for the configuration and pushing them to the Azure VM. This is where the new Azure VM DSC extension makes a difference.

With the release of [Azure VM DSC extension][2], the process of bootstrapping both LCM meta-configuration and other resource configuration becomes very easy. Let us see that process.

### Bootstrapping LCM meta-configuration

For this demonstration, I will use the following LCM meta-configuration script.

<pre class="brush: powershell; title: ; notranslate" title="">Configuration LCMConfiguration {
    Node Localhost {
       LocalConfigurationManager {
          RebootNodeIfNeeded = $true
       }
    }
}
</pre>

All we are doing in this configuration script is changing the _RebootNodeIfNeeded_ property. This can be enacted as a part of Azure VM creation using the method shown below.

```
#Set the Azure subscription
$SubscriptionName = "MyCloud"
Select-AzureSubscription -SubscriptionName $SubscriptionName

#Replace the variable values as needed
$ServiceName = "psdsc"
$Location = "Southeast Asia"
$VMName = "CSETest"
$StorageAccount = 'psdsc'
$StorageKey = 'storagekey'
$StorageContainer = 'dscarchives'

#Get the OS image reference
$ImageName = (Get-AzureVMImage | Where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | Select-Object -First 1).ImageName

#Create VM Config
$vmConfig = New-AzureVMConfig -Name $VMName -ImageName $ImageName -InstanceSize Small

#Create Provisioning Configuration
$vmProvisioningConfig = Add-AzureProvisioningConfig -VM $vmConfig -Windows -AdminUsername "AdminUser" -Password "P@ssw0rd"

$StorageContext = New-AzureStorageContext -StorageAccountName $StorageAccount -StorageAccountKey $StorageKey
Publish-AzureVMDscConfiguration -ConfigurationPath .\LCMConfiguration.ps1 -ContainerName $StorageContainer -StorageContext $StorageContext -Force

#Set the Azure VM DSC Extension to run the LCM meta-configuration
$vmAzureExtension = Set-AzureVMDscExtension -VM $vmProvisioningConfig -ConfigurationArchive LCMConfiguration.ps1.zip -ConfigurationName LCMConfiguration -Verbose -StorageContext $StorageContext -ContainerName $StorageContainer -Force

#Create a VM
New-AzureVM -ServiceName $ServiceName -VMs $vmAzureExtension
```

In the above script, the _Set-AzureVMDscExtension_ returns the VM object with the VM provisioning configuration specified using the _Add-AzureProvisioningConfig_ cmdlet. We finally provide that VM object to create a new Azure VM. Once the VM creation is complete, the VM DSC extension gets installed and the LCM meta-configuration gets enacted.

### Bootstrapping DSC Configuration

Bootstrapping DSC resource configuration is not any different from the process we just saw. Here is a sample configuration script that contains one custom DSC resource from the resource kit.


    Configuration AzureVMConfiguration {
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
 I saved the above configuration script as _AzureVMConfiguration.ps1_ and published the configuration using the _Publish-AzureVMDscConfiguration_ cmdlet. As I&#8217;d explained in an earlier article, when we publish a configuration script, any custom DSC resources imported using the _Import-DscResource_ cmdlet get packaged as an archive along with the configuration script and get uploaded to the storage account. This is the advantage of using VM DSC extension instead of custom script extension.

```
#Set the Azure subscription
$SubscriptionName = "MyCloud"
Select-AzureSubscription -SubscriptionName $SubscriptionName

#Replace the variable values as needed
$ServiceName = "psdsc"
$Location = "Southeast Asia"
$VMName = "CSETest"
$StorageAccount = 'psdsc'
$StorageKey = 'storagekey'
$StorageContainer = 'dscarchives'

#Get the OS image reference
$ImageName = (Get-AzureVMImage | Where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | Select-Object -First 1).ImageName

#Create VM Config
$vmConfig = New-AzureVMConfig -Name $VMName -ImageName $ImageName -InstanceSize Small

#Create Provisioning Configuration
$vmProvisioningConfig = Add-AzureProvisioningConfig -VM $vmConfig -Windows -AdminUsername "AdminUser" -Password "P@ssw0rd"

$StorageContext = New-AzureStorageContext -StorageAccountName $StorageAccount -StorageAccountKey $StorageKey
Publish-AzureVMDscConfiguration -ConfigurationPath .\AzureVMConfiguration.ps1 -ContainerName $StorageContainer -StorageContext $StorageContext -Force

#Set the Azure VM DSC Extension to run the LCM meta-configuration
$vmAzureExtension = Set-AzureVMDscExtension -VM $vmProvisioningConfig -ConfigurationArchive AzureVMConfiguration.ps1.zip -ConfigurationName AzureVMConfiguration -Verbose -StorageContext $StorageContext -ContainerName $StorageContainer -Force

#Create a VM
New-AzureVM -ServiceName $ServiceName -VMs $vmAzureExtension
```

The overall process of bootstrapping the DSC resource configuration along with the creation of a new Azure VM is shown above. At the end of this VM creation, you will have the DSC resources configured as per the configuration script. With this handy, there are many advanced scenarios we can enable with the help of DSC extension and a little bit of PowerShell!

[1]: /2014/07/29/bootstrapping-lcm-meta-configuration-in-a-azure-vm/
[2]: /2014/08/05/understanding-azure-vm-dsc-extension/