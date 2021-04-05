---
title: 'Infrastructure Blueprints – An Easier and Better Way to Share #PSDSC Configurations'
author: Ravikanth C
type: post
date: 2017-05-15T16:00:20+00:00
url: /2017/05/15/infrastructure-blueprints-a-way-to-share-psdsc-configurations/
views:
  - 10281
post_views_count:
  - 4347
categories:
  - DevOps
  - PowerShell DSC
tags:
  - DevOps
  - PowerShell DSC

---
A while ago, I wrote an article to [introduce Infrastructure Blueprints][1]. Today&#8217;s post is about a more refined version of that project. These infrastructure blueprints are the starting point for a bigger discussion on dynamic infrastructure. We will discuss that later. Read on!

Within the Infrastructure as Code (IaC) practices, there is enough emphasis on the repeatable and reusable automation enabled using configuration management tools. This is referred to as configuration as Code. Configuration as Code enables consistent methods to configure IT systems. And, when we integrate these processes into DevOps practices, we can ensure that the configuration across different stages of the deployment (Development / Test / Staging / Production) can be done in a efficient and consistent manner.

One of the best practices in Configuration as Code practice is to ensure that the configurations that we deploy are made reusable. This means that the configurations are parameterized and have the environmental data separated from the structural configuration data.

PowerShell Desired State Configuration (DSC) supports the separation of environmental configuration from structural configuration using the [configuration data][2] artifacts. And, sharing of parameterized configurations is done using the composite configuration modules or what we call [composite resources][3]. Composite configurations are very useful when a node configuration requires a combination multiple resources and becomes long and complex.

For example, building a Hyper-V cluster includes configuring host networking, domain join, updating firewall rules, creating/joining a cluster, and so on. Each node in the cluster should have this configuration done in a consistent manner. Therefore, the configurations that are applied for each of these items can be made reusable using the composite configuration methods.

Also, a real-world deployment pipeline implemented using IaC practices should also have validation of the infrastructure configuration at various stages of the deployment. In the PowerShell world, this is done using [Pester][4]. Within DSC too, Pester plays a very important role in validating the desired state after a configuration is applied and in the operations validation space.

### Infrastructure Blueprints

The [infrastructure blueprints][6] project provides guidelines on enabling reusable and repeatable DSC configurations combined with Pester validations that are identified by [Operations Validation Framework][7]. As a part of this repository, there will be a set of composite resource modules for various common configurations that you will see in a typical IT infrastructure.

Infrastructure Blueprints are essentially composite resource packages that contain node configurations and Pester tests that validate desired state and integration after the configuration is applied.

This repository contains multiple composite configuration modules. Each module contains multiple composite resources with ready to use examples and tests that validate the configuration.

The following folder structure shows how these composite modules are packaged.

![](/images/infrablue1.png)

  * _Diagnostics_ folder contains the Simple and Comprehensive tests for performing operations validation. 
      * **Simple**: A set of tests that validate the functionality of infrastructure at the desired state.
      * **Comprehensive**: A set of tests that perform a comprehensive operations validation of the infrastructure at the desired state.
      * For ease of identification, the test script names include _Simple_ or _Comprehensive_ within the file name.

The Operations Validation Framework can be used to retrieve the list of tests in this module and invoke the relevant ones.

![](/images/infrablue2.png)

Once you know the composite resource that is applied on the system, you can invoke either simple or comprehensive tests using the _Invoke-OperationValidation_ cmdlet.

![](/images/infrablue3.png)

  * _Examples_ folder contains a sample configuration data foreach composite configuration and also a configuration document that demonstrates how to use the composite resource.
  * _CompositeModuleName_.psd1 is the module manifest for the composite configuration module.

This manifest contains the _RequiredModules_ key that has all the required modules for the composite configuration to work. This is listed as a module specification object. For example, the _RequiredModules_ key for Hyper-VConfigurations composite module contains the following hashtable.

```powershell
# Modules that must be imported into the global environment prior to importing this module
RequiredModules = @(
    @{ModuleName='cHyper-v';ModuleVersion='3.0.0.0'},
    @{ModuleName='xNetworking';ModuleVersion='2.12.0.0'}
)
```


These composite modules are available in the PowerShell Gallery as well. And, therefore, having the _RequiredModules_ in the module manifest enables automatic download of all module dependencies automatically.

![](/images/infrablue4.png)

```powershell
PS C:\> Install-Module -Name Hyper-VConfigurations -Force -Verbose
VERBOSE: Using the provider 'PowerShellGet' for searching packages.
VERBOSE: The -Repository parameter was not specified.  PowerShellGet will use all of the registered repositories.
VERBOSE: Getting the provider object for the PackageManagement Provider 'NuGet'.
VERBOSE: The specified Location is 'https://www.powershellgallery.com/api/v2/' and PackageManagementProvider is 'NuGet'.
VERBOSE: Searching repository 'https://www.powershellgallery.com/api/v2/FindPackagesById()?id='Hyper-VConfigurations'' for ''.
VERBOSE: Total package yield:'1' for the specified package 'Hyper-VConfigurations'.
VERBOSE: Performing the operation "Install-Module" on target "Version '1.0.0.0' of module 'Hyper-VConfigurations'".
VERBOSE: The installation scope is specified to be 'AllUsers'.
VERBOSE: The specified module will be installed in 'C:\Program Files\WindowsPowerShell\Modules'.
VERBOSE: The specified Location is 'NuGet' and PackageManagementProvider is 'NuGet'.
VERBOSE: Downloading module 'Hyper-VConfigurations' with version '1.0.0.0' from the repository 'https://www.powershellgallery.com/api/v2/'.
VERBOSE: Searching repository 'https://www.powershellgallery.com/api/v2/FindPackagesById()?id='Hyper-VConfigurations'' for ''.
VERBOSE: Searching repository 'https://www.powershellgallery.com/api/v2/FindPackagesById()?id='cHyper-v'' for ''.
VERBOSE: Searching repository 'https://www.powershellgallery.com/api/v2/FindPackagesById()?id='xNetworking'' for ''.
VERBOSE: InstallPackage' - name='cHyper-V', version='3.0.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645'
VERBOSE: DownloadPackage' - name='cHyper-V', version='3.0.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645\cHyper-V\cHyper-V.nupkg', uri='https://www.powershe
llgallery.com/api/v2/package/cHyper-V/3.0.0'
VERBOSE: Downloading 'https://www.powershellgallery.com/api/v2/package/cHyper-V/3.0.0'.
VERBOSE: Completed downloading 'https://www.powershellgallery.com/api/v2/package/cHyper-V/3.0.0'.
VERBOSE: Completed downloading 'cHyper-V'.
VERBOSE: InstallPackageLocal' - name='cHyper-V', version='3.0.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645'
VERBOSE: InstallPackage' - name='xNetworking', version='3.2.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645'
VERBOSE: DownloadPackage' - name='xNetworking', version='3.2.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645\xNetworking\xNetworking.nupkg', uri='https://www
.powershellgallery.com/api/v2/package/xNetworking/3.2.0'
VERBOSE: Downloading 'https://www.powershellgallery.com/api/v2/package/xNetworking/3.2.0'.
VERBOSE: Completed downloading 'https://www.powershellgallery.com/api/v2/package/xNetworking/3.2.0'.
VERBOSE: Completed downloading 'xNetworking'.
VERBOSE: InstallPackageLocal' - name='xNetworking', version='3.2.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645'
VERBOSE: InstallPackage' - name='Hyper-VConfigurations', version='1.0.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645'
VERBOSE: DownloadPackage' - name='Hyper-VConfigurations', version='1.0.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645\Hyper-VConfigurations\Hyper-VConfigura
tions.nupkg', uri='https://www.powershellgallery.com/api/v2/package/Hyper-VConfigurations/1.0.0'
VERBOSE: Downloading 'https://www.powershellgallery.com/api/v2/package/Hyper-VConfigurations/1.0.0'.
VERBOSE: Completed downloading 'https://www.powershellgallery.com/api/v2/package/Hyper-VConfigurations/1.0.0'.
VERBOSE: Completed downloading 'Hyper-VConfigurations'.
VERBOSE: InstallPackageLocal' - name='Hyper-VConfigurations', version='1.0.0.0',destination='C:\Users\ravikanth_chaganti\AppData\Local\Temp\1037779645'
VERBOSE: Installing the dependency module 'cHyper-V' with version '3.0.0.0' for the module 'Hyper-VConfigurations'.
VERBOSE: Module 'cHyper-V' was installed successfully.
VERBOSE: Installing the dependency module 'xNetworking' with version '3.2.0.0' for the module 'Hyper-VConfigurations'.
VERBOSE: Module 'xNetworking' was installed successfully.
VERBOSE: Module 'Hyper-VConfigurations' was installed successfully.
```


As you can see in the above _Install-Module_ cmdlet output, the required modules are downloaded from the gallery. Thanks to [Chrissy][8] for this tip.

You can contribute to this project by submitting a pull request. All you need a set of composite resource modules packaged as a PowerShell module with DSC resources. Of course, ensure you add clear examples and tests.

[1]: http://www.powershellmagazine.com/2016/05/13/devops-infrastructure-as-code-and-powershell-dsc-infrastructure-blueprints/
[2]: https://msdn.microsoft.com/en-us/powershell/dsc/configdata
[3]: https://msdn.microsoft.com/en-us/powershell/dsc/authoringresourcecomposite
[4]: https://github.com/pester/pester
[5]: https://github.com/rchaganti/InfraBlueprints#infrastructure-blueprints
[6]: https://github.com/rchaganti/InfraBlueprints
[7]: https://github.com/PowerShell/Operation-Validation-Framework
[8]: https://blog.netnerds.net/2017/05/powershell-gallery-metapackages/