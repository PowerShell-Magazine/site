---
title: Cisco UCS Custom Resource for the Windows PowerShell Desired State Configuration (DSC)
author: Sumanth BR
type: post
date: 2017-05-29T16:00:59+00:00
url: /2017/05/29/cisco-ucs-custom-resource-for-the-windows-powershell-desired-state-configuration-dsc/
views:
  - 16143
post_views_count:
  - 5633
categories:
  - PowerShell DSC
tags:
  - Cisco
  - PowerShell DSC

---
The Cisco UCS PowerTool Suite is a set of PowerShell modules for Cisco UCS Manager, Cisco IMC (C-Series stand-alone servers) and Cisco UCS Central that helps in configuration and management of Cisco UCS domains and solutions.  The Cisco UCS PowerTool Suite 2.0.1 release added a new module **Cisco.Ucs.DesiredStateConfiguration** which consists of custom resources for configuring Cisco UCS Manager and Cisco IMC using the PowerShell DSC platform. You can download the latest version of the UCS PowerTool Suite from <a href="https://software.cisco.com/download/release.html?i=!y&mdfid=286305108&softwareid=284574017&release=2.0.2" target="_blank" rel="noopener noreferrer">cisco.com</a>. Refer to <a href="https://communities.cisco.com/docs/DOC-37154" target="_blank" rel="noopener noreferrer">Cisco UCS PowerTool Suite</a> page on Cisco Communities for more resources.

PowerShell Desired State Configuration (DSC) is a management platform which enables you to configure, deploy, and manage systems. DSC provides **declarative**, **autonomous** and **idempotent** deployment, configuration and conformance for standards-based managed elements. For more information on DSC refer to the <a href="https://msdn.microsoft.com/en-us/powershell/dsc/overview" target="_blank" rel="noopener noreferrer">PowerShell DSC documentation</a>.

Cisco UCS DSC Resource aids in achieving Configuration as Code in turn helping you to follow the DevOps model.

The Cisco UCS DSC module provides six DSC custom resources which cover the majority of use cases.  You can view the custom UCS resources by running the _Get-DscResource_ cmdlet as shown below.

![](/images/cisco1.png)

### Solution Architecture Overview

Before getting in to the details of the resources let’s review some basic concepts of DSC and the overall architecture of the UCS PowerTool DSC solution.

The DSC management platform consists of three main components.

  * **Configuration:** This is where you define the configurations that need to be applied in a declarative manner. Once you run this configuration, DSC will take care of ensuring the system is in the state that is defined in the configuration.
  * **Resources:** These are the building blocks for the configurations.
  * **Local Configuration Manager (LCM):** This is the engine that facilitates the interaction between resources and configurations. The LCM regularly polls the state of the system and takes appropriate actions based on the resource. The LCM runs on every target node.

In DSC there are two ways to deploy a configuration.

  * **Push Mode** &#8211; Push mode is a manual deployment of DSC resources to target nodes. Once the configuration is compiled the user runs the Start-DscConfiguration cmdlet to push the configuration to the desired nodes.
  * **Pull Mode** &#8211; Pull mode configures nodes to check in to a DSC web pull server to retrieve the configuration files. When a new configuration is available, the LCM downloads and applies it to the target node.

To utilize DSC functionality with Cisco UCS Manager or Cisco IMC an intermediate server is required. The intermediate server is a Windows Server having the required Windows Management Framework (WMF), PowerShell and the UCS PowerTool Suite installed.  A typical architecture is shown in the figure.

![](/images/cisco2.jpg)

**Central Server**—This server is used to write the UCS DSC configuration scripts for Cisco UCS Manager or Cisco IMC. This can be configured as a pull server if the method of deployment is **Pull Mode**.

**Intermediate Server**— The Central server deploys the configuration to the Intermediate server. This server applies the configuration to the Cisco UCS Manager or Cisco IMC using the UCS PowerTool DSC cmdlets.

## Cisco UCS Manager DSC Resources

There are four custom resources provided for configuring Cisco UCS Manager.

  * **UcsSyncMoWithReference:** This resource syncs configuration from a reference UCS Manager to any target UCS Managers.
  * **UcsManagedObject:** This resource configures any UCS Manager Managed Object (MO) by specifying the MO details.
  * **UcsScript:** This resource allows for the execution of UCS Manager PowerTool cmdlets.
  * **UcsSyncFromBackup:** This resource applies configuration from a backup file to any target UCS Managers.

### Generating DSC configuration document for UCS Manager GUI operations

To simplify the process of authoring the DSC configuration documents for UCS Manager use the **ConvertTo-UcsDscConfig** cmdlet. This cmdlet is similar to the **ConvertTo-UcsCmdlet** cmdlet that generates the UCS PowerTool cmdlets for the actions performed on the UCS Manager GUI.  Creating a configuration document is a simple two-step process.

  1. Launch the UCS Manager GUI, either using the **Start-UcsGuiSession** cmdlet or manually, then run the **ConvertTo-UcsDscConfig** cmdlet, if -OutputFilePath is supplied the DSC configuration will be written to the specified file, otherwise the ConvertTo-UcsDscConfig cmdlet will produce output to the UCS PowerTool console session.
  2. Create the required configuration using the UCS Manager GUI. When you are done creating configurations you can open the input file specified in the step 1. The required DSC configuration document will have been auto-generated. If no output file was specified, cut and paste the UCS PowerTool console output to a file.

Once you have the auto-generated document you just need to customize a few environment-related settings like Configuration Data, UCS Manager connection details and Credentials.

### UcsSyncMoWithReference Custom Resource

If you have more than one UCS domain in your datacenter and want to maintain a baseline configuration across all the UCS domains use the UcsSyncMoWithReference resource.

You can create a configuration using this resource by specifying the Distinguished Name (DN) of the Managed Object (MO) that needs to be synced.  Here is an example of how you can sync a Service Profile, Service Profile Template, and LDAP Groups.

```powershell
UcsSyncMoWithReference SyncServiceProfile
{
    UcsCredentials = $ucsCredential
    UcsConnectionString = $ucsConnString
    RefUcsCredentials = $refUcsCredential
    RefUcsConnectionString = $refUcsConString
    Ensure="Present"
    Identifier ="2"
    Hierarchy=$true
    Dn = "org-root/ls-SPExchangeServer"
} 

UcsSyncMoWithReference SyncSpTemplate
{
    UcsCredentials = $ucsCredential
    UcsConnectionString = $ucsConnString
    RefUcsCredentials = $refUcsCredential
    RefUcsConnectionString = $refUcsConString
    Ensure="Present"
    Identifier ="3"
    Hierarchy=$true
    Dn = "org-root/ls-SPTSqlServer"
}

UcsSyncMoWithReference SyncLDAPGroups
{
    UcsCredentials = $ucsCredential
    UcsConnectionString = $ucsConnString
    RefUcsCredentials = $RefUcsCredential
    RefUcsConnectionString = $refUcsConString
    Ensure="Present"
    Identifier ="4"
    DeleteNotPresent=$true

     #Sync all the LDAP groups by specifying the DN and Hierarchy true
     Hierarchy=$true
     Dn="sys/ldap-ext"        
 }
```

In the above example I have specified the DN of the SP, SP Template, and the LDAP group. By specifying Ensure=&#8221;Present&#8221; the UcsSyncMoWithReference resource ensures that the MOs are created on the UCS domain. You can also specify what action to take in case if there are additional MOs than compared to the MOs present on the reference UCS. If you want to delete the additional MOs, you need to specify DeleteNotPresent= $true as done in the LDAP sync configuration in the above example.  Refer to UCS Manager PowerTool User Guide for more details on the properties of the Resource.

### UcsManagedObject Custom Resource

This is a generic resource provided to configure any MO in UCS Manager. To use this resource, you need to be familiar with the MO definitions and properties. One way to make use of this resource is by generating this configuration automatically as explained in the earlier section.  If you are writing the configuration manually refer to the [UCS Manager XML API Programmer’s Guide][1].  Below is an example configuration of creating an Org in the UCS Manager. There are few key things to consider while creating the configuration, you need to specify the DN, XML API Class ID, and the Property Map.

```powershell
UcsManagedObject CreateOrganisationDemo
{
    Ensure = "Present"
    ModifyPresent = $true
    ClassId= "orgOrg"
    Dn = "org-root/org-DSCDemoOrg"
    PropertyMap= "Descr = test for DSC with certificate `nName = DSCDemoOrg"
    UcsCredentials = $ucsCredential
    UcsConnectionString = $connectionString
    Identifier = "2"
}
```


You need to specify the properties of the managed object as key value pairs using the below format

`<key1>=<value1> <key>=<value2>`

### UcsScript Custom Resource

This is a generic resource provided to execute UCS Manager PowerTool cmdlets in a script. You can use this resource in cases where the configuration is complex and needs to be written as a script. You can also generate the configuration automatically for this resource as explained earlier.  Below is an example configuration of renaming a Service Profile.

```powershell
UcsScript RenameServiceProfileDemo
{
    Ensure = "Present"
    Dn = "org-root/ls-dscdemo"
    Script = "Get-UcsOrg -Level root | Get-UcsServiceProfile -Name 'TestSP' -LimitScope | Rename-UcsServiceProfile -NewName 'dscdemo' "
    UcsCredentials = $ucsCredential
    UcsConnectionString = $connectionString
    Identifier ="1"
}
```


If the configuration script is complex you can specify multiple DNs in a comma separated format.

### Configuration Example

This section details how you can put together all the things in a DSC configuration document.

For all the examples mentioned above you need to specify environment settings, UCS Connection details and Credentials.

UCS connection string needs to be specified in the following format.

```powershell
Name=<ipAddress> [`nNoSsl=<bool>][`nPort=<ushort>] [`nProxyAddress=<proxyAddress>] [`nUseProxyDefaultCredentials=<bool>]
```

UCS Manager credentials needs to be specified as a PSCredential object. You can use certificates for encrypting the credentials to keep it secure. For information on using certificates for encryption refer to <a href="https://msdn.microsoft.com/en-us/powershell/dsc/securemof" target="_blank" rel="noopener noreferrer">Microsoft DSC documentation</a>.

Below is an example configuration which has all the details.

```powershell
$ConfigData=    @{
        AllNodes = @(    
                        @{ 
                            # The name of the node we are describing
                            NodeName ="10.105.219.128"
                            # The path to the .cer file containing the
                            # public key of the Encryption Certificate
                            # used to encrypt credentials for this node
                             CertificateFile = "C:\Certificate\MyCertificate.cer"


                            # The thumbprint of the Encryption Certificate
                            # used to decrypt the credentials on target node
                            Thumbprint = "558CF40844CDC6303D25494FB007189F75BEE060"
                        };
                    );   
} 

Configuration AutoGeneratedConfig
{
    param
    (
        [Parameter(Mandatory=$true)]
        [PsCredential] $ucsCredential,

        [Parameter(Mandatory=$true)]
        [string] $connectionString
    )
    
        Import-DSCResource -ModuleName Cisco.Ucs.DesiredStateConfiguration
  
      Node "10.105.219.128"
      {   
            LocalConfigurationManager
            {
                CertificateId = $node.Thumbprint
                ConfigurationMode = 'ApplyOnly'
               RefreshMode = 'Push'
               
            } 
    
            UcsManagedObject UcsManagedObject1
            {
                Ensure = "Present"
                ModifyPresent = $true
                ClassId= "equipmentLocatorLed"
                Dn = "sys/chassis-1/blade-1/locator-led"
                PropertyMap= "Id = 1 `nBoardType = single `nAdminState = on"
                UcsCredentials = $ucsCredential
                UcsConnectionString = $connectionString
                Identifier = "1"
            }


           UcsManagedObject ucsManagedobject2
           {
               Ensure = "Present"
               ModifyPresent = $true
               ClassId= "orgOrg"
               Dn = "org-root/org-SubOrg2"
               PropertyMap= "Descr = test for DSC with certificate `nName = SubOrg2"
               UcsCredentials = $ucsCredential
               UcsConnectionString = $connectionString
               Identifier = "2"
           }
      }
 }

try
{
	${Error}.Clear()
    $credential = Get-Credential

    AutoGeneratedConfig  -ConfigurationData $ConfigData `
                         -ucsCredential $credential `
                         -connectionString "Name=10.65.183.5" `
                         -OutputPath "C:\DscDemo\AutoGeneratedConfig"  
}
Catch
{
    Write-Host ${Error}
    exit
}
```

When you run this script PowerShell will generate the corresponding MOF files. You can deploy the configuration based on the LCM configuration mode. If it is set to Push mode, then you can enact the configuration using the below syntax.

```powershell
Start-DscConfiguration -Path "C:\DscDemo\AutoGeneratedConfig\" -Wait -Verbose -force
```

[1]: http://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/sw/api/b_ucs_api_book.html