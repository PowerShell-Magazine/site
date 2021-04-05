---
title: Deploy Custom Azure Automation Integration Modules Using ARM Templates
author: Ravikanth C
type: post
date: 2015-11-26T17:00:06+00:00
url: /2015/11/26/deploy-custom-azure-automation-integration-modules-using-arm-templates/
views:
  - 17160
post_views_count:
  - 4169
categories:
  - Azure
  - Azure Resource Manager
  - Azure Automation
tags:
  - Azure
  - Azure Automation
  - Azure Resource Manager

---
**Update:** Earlier generic linked template for deploying modules was removed and it was breaking my ARM template. I published two linked templates in my ARMSeries repo that can be used to deploy modules to Azure Automation accounts.

Azure Automation has the integration module concept for extending what you can do using the runbooks. These [modules are PowerShell modules][1] and can contain functions or workflows. [This article][1] from Azure team blog explains how to author these custom integration modules. So, I won&#8217;t repeat that here and safely assume that you have a module that you want to deploy to Azure Automation. You can follow this article and try deploying the demo module I built for this purpose.

You can import these integration modules into Azure Automation using any of the following methods.

  1. Using the _New-AzureRmAutomationModule_ cmdlet in the AzureRm.Automation module.
  2. Using the preview portal and navigating to the Assets within automation account.
  3. Using Azure Resource Manager (ARM) template

The first two methods are straightforward. I will show you the ARM template way of doing this. Btw, this is not new and has been there for a while. If you look at some of the modules on PowerShell Gallery, you will see an option to &#8220;Deploy to Azure Automation&#8221;.

![](/images/aadeploy1.png)

When you click on this button within the PowerShell Gallery page for the module, an ARM template gets deployed. In this article, I will show you how to build this ARM template yourself and deploy your custom integration modules so that you can make this a part of larger ARM template deployment process. To get started, here is the ARM template to deploy [my custom module][2]. This module is just for demonstration purpose and it has no useful activities.

```json
{
  "$schema": "http://schemas.microsoft.org/azure/deploymentTemplate?api-version=2015-01-01#",
  "contentVersion": "1.0",
  "parameters": {
    "New or existing Automation account": {
      "type": "string",
      "allowedValues": [
        "New",
        "Existing"
      ],
      "metadata": {
        "description": "Select whether you want to create a new Automation account or use an existing account. WARNING: if you select NEW but use an Automation account name that already exists in your subscription, you will not be notified that your account is being updated. The pricing tier for the account will be set to free and any tags on the account will be erased."
      }
    },
    "Automation Account Name": {
      "type": "string",
      "metadata": {
        "description": "The module will be imported to this Automation account. If you want to import your module to an existing account, make sure the resource group matches and you have entered the correct name. The account name must be between 6 to 50 characters, and can contain only letters, numbers, and hyphens."
      }
    },
    "Automation Account Location": {
      "type": "string",
      "metadata": {
        "description": "The location to deploy the Automation account in. If you select an existing account, the location field will not be used."
      }
    },
    "ModuleName" : {
      "type" : "String",
      "metadata" : {
        "description" : "Name of the module being imported into the Azure Automation account."
      }
    },
    "ModuleUri" : {
      "type" : "string",
      "metadata" : {
          "description" : "URI for the AA module zip archive. This must be accessible from automation account."
      }
    }
  },
  "variables": {
    "templatelink": "[concat('https://raw.githubusercontent.com/rchaganti/armseries/master/', parameters('New or Existing Automation account'), 'AccountTemplate.json')]"
  },
  "resources": [
    {
      "apiVersion": "2015-01-01",
      "name": "nestedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
        "mode": "incremental",
        "templateLink": {
          "uri": "[variables('templatelink')]",
          "contentVersion": "1.0"
        },
        "parameters": {
          "accountName": {
            "value": "[parameters('Automation Account Name')]"
          },
          "accountLocation": {
            "value": "[parameters('Automation Account Location')]"
          },
          "ModuleName": {
            "value": "[parameters('ModuleName')]"
          },
          "ModuleUri": {
            "value": "[parameters('ModuleUri')]"
          }
        }
      }
    }
  ],
  "outputs": {}
}
```


Within this ARM template, we use parameters for collecting information about the Automation account. If the provided Automation account name is a new one, we will create it first; if it&#8217;s the existing one, we can simply publish the integration module. ModuleUri can be any location where the integration module zip file is stored. This location should be accessible to the service deploying the integration module.

If you notice, we are using a [linked template deployment][3]. Using linked template deployments, we can decompose the ARM template into multiple small chunks that are purpose-specific templates. In the template above, I specified the linked template link in line number 23. This is a template hosted by Microsoft and the JSON template file name changes based on the automation account type we select during deployment.

Let us see how this works! I will use the cmdlets in Azure PowerShell module to deploy this template. First authenticate to Azure using the _Login-AzureRmAccount_ cmdlet.

We will now validate the above template and ensure everything is fine before we deploy it. For running the following command, you will need an existing resource group.

```powershell
$parameters = @{
    'moduleName' = 'myModule'
    'moduleUri' = 'https://github.com/rchaganti/armseries/raw/master/MyModule.zip'
    'Automation Account Name' = 'fuarmdemo'
    'Automation Account Type' = 'Existing'
    'Automation Account Location' = 'eastus2'
    'TemplateFile' = 'C:\Temp\DeployModule.json'
}
Test-AzureRmResourceGroupDeployment @parameters -Verbose
```


Once we see that the template is valid, we can go ahead and deploy it using the _New-AzureRmResourceGroupDeployment_ cmdlet.

```powershell
New-AzureRmResourceGroupDeployment @parameters -Verbose
```

Once this deployment is complete, you will see output similar to this.

![](/images/aadeploy2.png)

You can navigate to the Automation account on the Azure preview portal and see that the module is indeed present in the Assets blade.

![](/images/aadeploy3.png)

This is it. You can use this template and make it a part of a larger ARM deployment using nested or linked template approach.

[1]: https://azure.microsoft.com/en-us/blog/authoring-integration-modules-for-azure-automation/
[2]: https://github.com/rchaganti/armseries/blob/master/MyModule.zip
[3]: https://azure.microsoft.com/en-us/documentation/articles/resource-group-linked-templates/