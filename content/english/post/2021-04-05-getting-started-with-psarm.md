---
title: Getting Started with PSArm
author: Ravikanth C
type: post
date: 2021-04-05
url: /2021/04/05/getting-started-with-arm/
images:
  - "images/posts/psarm.png"
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

In the first part of this [series](/tags/psarm), you learned about [PSArm](https://github.com/powershell/psarm) -- a PowerShell embedded DSL -- that you can use to declaratively define your Azure infrastructure and generate an ARM template. PSArm, being a PowerShell DSL, supports PowerShell language constructs and builds the object model for ARM templates on top of PowerShell. So, if you are already familiar with PowerShell, writing ARM templates now will be as simple as writing another PowerShell script. So, how do you get started?

### Installing PSArm

You can get PSArm module from the [PowerShell gallery](https://www.powershellgallery.com/packages/PSArm).

```powershell
Install-Module -Name PSArm -AllowPrerelease
```

You can also build module from the [source code available on GitHub](https://github.com/powershell/psarm). 

This module, when loaded, exports a bunch of functions and cmdlets.

```powershell
PS C:\sandbox\psarm> Get-Command -Module PSArm
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------                 
Function        add                                                0.1.0      PSArm             
Function        and                                                0.1.0      PSArm          
Function        array                                              0.1.0      PSArm
Function        base64                                             0.1.0      PSArm
Function        base64ToJson                                       0.1.0      PSArm
Function        base64ToString                                     0.1.0      PSArm
Function        bool                                               0.1.0      PSArm
Function        coalesce                                           0.1.0      PSArm
Function        concat                                             0.1.0      PSArm
Function        contains                                           0.1.0      PSArm
Function        copyIndex                                          0.1.0      PSArm
Function        createArray                                        0.1.0      PSArm
Function        createObject                                       0.1.0      PSArm
Function        dataUri                                            0.1.0      PSArm
Function        dataUriToString                                    0.1.0      PSArm
Function        dateTimeAdd                                        0.1.0      PSArm
Function        deployment                                         0.1.0      PSArm
Function        div                                                0.1.0      PSArm
Function        empty                                              0.1.0      PSArm
Function        endsWith                                           0.1.0      PSArm
Function        environment                                        0.1.0      PSArm
Function        equals                                             0.1.0      PSArm
Function        extensionResourceId                                0.1.0      PSArm
Function        false                                              0.1.0      PSArm
Function        first                                              0.1.0      PSArm
Function        float                                              0.1.0      PSArm
Function        format                                             0.1.0      PSArm
Function        greater                                            0.1.0      PSArm
Function        greaterOrEquals                                    0.1.0      PSArm
Function        guid                                               0.1.0      PSArm
Function        if                                                 0.1.0      PSArm
Function        indexOf                                            0.1.0      PSArm
Function        int                                                0.1.0      PSArm
Function        intersection                                       0.1.0      PSArm
Function        json                                               0.1.0      PSArm
Function        last                                               0.1.0      PSArm
Function        lastIndexOf                                        0.1.0      PSArm
Function        length                                             0.1.0      PSArm
Function        less                                               0.1.0      PSArm
Function        lessOrEquals                                       0.1.0      PSArm
Function        list                                               0.1.0      PSArm
Function        listAccountSas                                     0.1.0      PSArm
Function        listAdminKeys                                      0.1.0      PSArm
Function        listAuthKeys                                       0.1.0      PSArm
Function        listChannelWithKeys                                0.1.0      PSArm
Function        listClusterAdminCredential                         0.1.0      PSArm
Function        listConnectionStrings                              0.1.0      PSArm
Function        listCredential                                     0.1.0      PSArm
Function        listCredentials                                    0.1.0      PSArm
Function        listKeys                                           0.1.0      PSArm
Function        listKeyValue                                       0.1.0      PSArm
Function        listPackage                                        0.1.0      PSArm
Function        listQueryKeys                                      0.1.0      PSArm
Function        listRawCallbackUrl                                 0.1.0      PSArm
Function        listSecrets                                        0.1.0      PSArm
Function        listServiceSas                                     0.1.0      PSArm
Function        listSyncFunctionTriggerStatus                      0.1.0      PSArm
Function        max                                                0.1.0      PSArm
Function        min                                                0.1.0      PSArm
Function        mod                                                0.1.0      PSArm
Function        mul                                                0.1.0      PSArm
Function        newGuid                                            0.1.0      PSArm
Function        not                                                0.1.0      PSArm
Function        null                                               0.1.0      PSArm
Function        or                                                 0.1.0      PSArm
Function        padLeft                                            0.1.0      PSArm
Function        parameters                                         0.1.0      PSArm
Function        providers                                          0.1.0      PSArm
Function        range                                              0.1.0      PSArm
Function        reference                                          0.1.0      PSArm
Function        replace                                            0.1.0      PSArm
Function        resourceGroup                                      0.1.0      PSArm
Function        resourceId                                         0.1.0      PSArm
Function        skip                                               0.1.0      PSArm
Function        split                                              0.1.0      PSArm
Function        startsWith                                         0.1.0      PSArm
Function        string                                             0.1.0      PSArm
Function        sub                                                0.1.0      PSArm
Function        subscription                                       0.1.0      PSArm
Function        subscriptionResourceId                             0.1.0      PSArm
Function        substring                                          0.1.0      PSArm
Function        take                                               0.1.0      PSArm
Function        tenantResourceId                                   0.1.0      PSArm
Function        toLower                                            0.1.0      PSArm
Function        toUpper                                            0.1.0      PSArm
Function        trim                                               0.1.0      PSArm
Function        true                                               0.1.0      PSArm
Function        union                                              0.1.0      PSArm
Function        uniqueString                                       0.1.0      PSArm
Function        uri                                                0.1.0      PSArm
Function        uriComponent                                       0.1.0      PSArm
Function        uriComponentToString                               0.1.0      PSArm
Function        utcNow                                             0.1.0      PSArm
Function        variables                                          0.1.0      PSArm
Cmdlet          ConvertFrom-ArmTemplate                            0.1.0      PSArm
Cmdlet          ConvertTo-PSArm                                    0.1.0      PSArm
Cmdlet          New-PSArmDependsOn                                 0.1.0      PSArm
Cmdlet          New-PSArmEntry                                     0.1.0      PSArm
Cmdlet          New-PSArmFunctionCall                              0.1.0      PSArm
Cmdlet          New-PSArmOutput                                    0.1.0      PSArm
Cmdlet          New-PSArmResource                                  0.1.0      PSArm                   
Cmdlet          New-PSArmSku                                       0.1.0      PSArm
Cmdlet          New-PSArmTemplate                                  0.1.0      PSArm
Cmdlet          Publish-PSArmTemplate                              0.1.0      PSArm 
```

You can see that the exported functions are similar to what ARM template language offers. 

### PSArm Syntax Basics 

A typical PSArm script for ARM template starts with the `Arm` keyword. Within the `Arm` body, you define each Azure resource using the `Resource` keyword. 

```powershell
Arm {
    Resource "identifier" -Namespace "Microsoft.resource" -ApiVersion "resourceApiVersion" -Type "resourceType" {
        properties {
            "propertyName" "value"
        }
    }

    output "Output from deployment"
}
```

The `Arm`, `Resource`, and `output` keywords are aliases defined in the module.

```powershell
PS C:\sandbox\psarm> (Get-Alias).Where({$_.Source -eq 'PSArm'})
CommandType     Name                                               Version    Source    
-----------     ----                                               -------    ------                 
Alias           Arm -> New-PSArmTemplate                           0.1.0      PSArm
Alias           ArmArray -> New-PSArmArray                         0.1.0      PSArm
Alias           ArmElement -> New-PSArmElement                     0.1.0      PSArm
Alias           ArmSku -> New-PSArmSku                             0.1.0      PSArm
Alias           DependsOn -> New-PSArmDependsOn                    0.1.0      PSArm
Alias           Output -> New-PSArmOutput                          0.1.0      PSArm
Alias           RawCall -> New-PSArmFunctionCall                   0.1.0      PSArm
Alias           RawEntry -> New-PSArmEntry                         0.1.0      PSArm
Alias           Resource -> New-PSArmResource                      0.1.0      PSArm
```

With `Arm` keyword, you can specify an optional `Name` parameter which will be used as a name of the deployment within the template. For the resource definition, you use the `Resource` keyword. You must specify a `Name` to be used for the resource you want to provision, `Namespace` of the resource, `ApiVersion`, and `Type`.  As you enter arguments for these four parameters, you will see that PowerShell dynamically adds some more parameters based on the type of resource you intend to provision.

![](/images/psarmresourceparam.png)

For example, as you see in the above screenshot, as soon as I added the `Type` parameter and its argument, I get `Kind` and `Tags` as the dynamic parameters. You can, then, use the `ArmSku` keyword to specify the SKU of the resource that you intend to provision. In case of a storage account, this can be set to Standard_LRS or any other supported value. Each Azure resource may need some more additional properties for resource provisioning and configuration. You can use the `properties` keyword for this purpose. PSArm gives you the auto-completion of property names within the `properties` block.

![](/images/psaramresourceprop.png)

Finally, the `output` keyword can be used to retrieve properties required from the deployed resource objects. This keyword takes `Name`, `Type`, and `Value` as parameters.

### First PSArm Script

Here is a complete PSArm script for provisioning a simple storage account.

```powershell
Arm -Name myFirstTemplate -Body {
    Resource 'mypsarmsaccount' `
             -Namespace 'Microsoft.Storage' `
             -Type 'storageAccounts' `
             -apiVersion '2019-06-01' `
             -Kind 'StorageV2' `
             -Location 'WestUS' {
        ArmSku 'Standard_LRS'
        Properties {
            accessTier 'Hot'
        }
    }

    Output 'storageResourceId' -Type 'string' -Value (ResourceId 'Microsoft.Storage/storageAccounts' (Concat 'myPsArmSaccount'))
}
```

Make a note of the `ResourceId` and `Concat` functions used along with the `output` keyword. PSArm has public function parity with what is offered in ARM template language. You can save this script with a `.psarm.ps1` extension and generate the ARM template JSON using the `Publish-PSArmTemplate` cmdlet that PSArm module provides.

```powershell
Publish-PSArmTemplate -Path .\firstTemplate.psarm.ps1 `
                      -OutFile .\firstTemplate.json -Force
```

Here is the generated ARM template JSON for the PSArm script that you just built.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "_generator": {
      "name": "psarm",
      "version": "0.1.0.0",
      "psarm-psversion": "5.1.19041.610",
      "templateHash": "5716551932369025750"
    }
  },
  "resources": [
    {
      "name": "myFirstTemplate",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "name": "mypsarmsaccount",
              "apiVersion": "2019-06-01",
              "type": "Microsoft.Storage/storageAccounts",
              "kind": "StorageV2",
              "location": "WestUS",
              "sku": {
                "name": "Standard_LRS"
              },
              "properties": {
                "accessTier": "Hot"
              }
            }
          ],
          "outputs": {
            "storageResourceId": {
              "type": "string",
              "value": "[resourceId('Microsoft.Storage/storageAccounts', concat('myPsArmSaccount'))]"
            }
          }
        }
      }
    }
  ]
}
```

This ARM template can be deployed using your favorite command line tool or using template deployment in Azure Portal.

When using Azure CLI,

```shell
az deployment group create --resource-group psarm --template-file .\firstTemplate.json
```

When using Azure PowerShell,

```powershell
New-AzResourceGroupDeployment -ResourceGroupName psarm -TemplateFile .\firstTemplate.json
```

This is it. Congratulations. You just used PowerShell based DSL to generate and deploy an ARM template. In the next part of this series, you will learn more parameterizing PSArm scripts.