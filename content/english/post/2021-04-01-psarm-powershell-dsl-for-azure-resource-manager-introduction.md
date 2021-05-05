---
title: PSArm - PowerShell DSL For Azure Resource Manager - Introduction
author: Ravikanth C
type: epic
date: 2021-04-01
url: /2021/04/01/psarm-powershell-dsl-for-azure-resource-manager-introduction/
images:
  - "images/posts/psarm.png"
post_views_count:
  - 1790
views:
  - 1617
categories:
  - Azure Resource Manager
  - PSArm
  - Azure
  - Module Spotlight
tags:
  - Modules
  - PSArm
  - Azure Resource Manager
  - Azure
---

Those who worked on Azure Resource Manager (ARM) templates understand that complexity of writing and making sure the template works as expected becomes very complicated as the complexity of a deployment grows. ARM template language uses JSON data representation format and it is not a language to be honest. It is fragments of functions and other programming constructs embedded into JSON which only the Azure Resource Manager understands. There are tools such as the Visual Studio Code extension that attempt to simplify the ARM template authoring experience. But, end of the day, when you have to debug an issue within the ARM template, it won't be easy based on the complexity of the template. There are other tools that intend to simplify provisioning Azure resources.

[HashiCorp Terraform](https://www.terraform.io/) is one of the most popular provisioning tools that can provision Azure resources. [Pulumi](https://www.pulumi.com/) provides the necessary support to define your Azure infrastructure in a supported programming language like write Go, Python, C#, F#, and so on. Both Terraform and Pulumi support multiple cloud providers. You may have recently seen or read about [Project Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/bicep-overview) as well. Bicep is domain-specific language meant for generating ARM templates. The most recent entrant in this race is what the PowerShell product team announced -- [PSArm module](https://devblogs.microsoft.com/powershell/announcing-the-preview-of-psarm/) (preview) which provides a PowerShell-embedded domain-specific language (DSL) for Azure Resource Manager (ARM) templates. This is an [experimental module](https://github.com/powershell/psarm) and certainly not meant for production use yet. This module enables users to leverage the existing PowerShell scripting knowledge to author ARM templates using PowerShell. Once you write ARM template as a PowerShell DSC using [PSArm](https://www.powershellgallery.com/packages/PSArm) which can be used to generate the ARM template JSON. This, at the moment, is just an experimental project and more like a proof-of-concept for creating a DSL within PowerShell. 

Comparison between Project Bicep and PSArm naturally arises as both these projects are from Microsoft.

### Project Bicep

Bicep is a domain-specific language (DSL) that is specifically designed for transpiling Bicep code into ARM templates. Generating ARM templates is the only purpose here. Bicep is not a general-purpose programming language. At the time of this writing, Bicep is in the very early stages of development but it is supported by Microsoft for production use. Think of Bicep as a transparent abstraction on Azure Resource Manager. For someone who is getting started with Azure Resource Manager and Azure deployments, Bicep will be a great start. It is simple and easy to learn. No doubt. Bicep can even decompile (on a best effort basis) your existing ARM templates to Bicep. Bicep seems to have taken some inspiration from how Terraform use HashiCorp Configuration Language (HCL). Bicep already  has an excellent VS Code extension that makes your life easy when authoring Bicep files.

Here is a Bicep file looks like. This is a simple storage account creation.

```
param storageAccountName string

@allowed([
  'Hot', 
  'Cool',
  'Archive'
])
@
param accessTier string = 'Hot'

@allowed([
  'WestUS2',
  'CentralUS'
])
param location string = 'WestUS2'

resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

You can build an ARM template from this Bicep file using the bicep command line.

```shell
bicep build main.bicep
```

But since Bicep is a language meant only for ARM template generation, there may not be a way to interact with other parts of the system. For example, what if you had certain configuration artifacts stored in a CMDB and you want to pull that data and use in a Bicep program. This won't be possible. At least for now. You will be restricted to what language constructs are available within Bicep.

### PSArm

PSArm on the other hand is an embedded DSL module. If you are already familiar with PowerShell language, it will be quite natural to choose a DSL that works in PowerShell. There is no need to learn another language. With PowerShell, you get access to a wide-range of APIs other than what PSArm may provide. This helps in creating complex and dynamic ARM templates with just what PowerShell can do. You get use all aspects of PowerShell language and the underlying infrastructure. 

Here is how a PSArm script looks like. Again, this is for simple storage account creation.

```powershell
param(
    [Parameter(Mandatory)]
    [string]
    $StorageAccountName,

    [Parameter()]
    [ValidateSet('WestUS2', 'CentralUS')]
    [string]
    $Location = 'WestUS2',

    [Parameter()]
    [ValidateSet('Hot', 'Cool', 'Archive')]
    [string]
    $AccessTier = 'Hot'
)

Arm {
    Resource $StorageAccountName -Namespace 'Microsoft.Storage' -Type 'storageAccounts' -apiVersion '2019-06-01' -Location $Location {
        ArmSku 'Standard_LRS'
        Properties {
            accessTier $AccessTier
        }
    }
}
```

Here is how you can compile the above PSArm file into an ARM template.

```powershell
Publish-PSArmTemplate -Path .\newStorageAccount.psarm.ps1 -Parameters @{
    storageAccountName = 'storageName'
    location = 'location'
}
```

You can see that both Bicep and PSArm files more or less have the same number of lines. PowerShell is a little more verbose given the nature of how PowerShell commands are built. 

At this point in time, I don't see this as a Bicep vs PSArm. It is about whether you want to learn a new language or use your existing skills. This is just a quick introduction to PSArm. Next in this [series of articles on PSArm](/tags/psarm), you will learn more about using PSArm. 