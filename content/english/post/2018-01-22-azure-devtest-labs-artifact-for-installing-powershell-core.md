---
title: Azure DevTest Labs Artifact for installing PowerShell Core
author: Ravikanth C
type: post
date: 2018-01-22T17:00:50+00:00
url: /2018/01/22/azure-devtest-labs-artifact-for-installing-powershell-core/
views:
  - 10653
post_views_count:
  - 5235
categories:
  - Azure
  - Azure DevTest
tags:
  - Azure
  - Azure DevTest

---
Unless you were living under a rock, [PowerShell Core][1] [general availability][2] isn&#8217;t any breaking news.

> PowerShell Core is a cross-platform (Windows, Linux, and macOS) automation and configuration tool/framework that works well with your existing tools and is optimized for dealing with structured data (e.g. JSON, CSV, XML, etc.), REST APIs, and object models. It includes a command-line shell, an associated scripting language and a framework for processing cmdlets.

Some time in 2016, I published the [Azure DevTest Labs artifact for installing PowerShell for Linux][3] on Azure Linux virtual machines. Similar to this, I have now created a new artifact for installing PowerShell Core on Windows VMs in Azure. This new artifact is still not in the official artifacts repository and it is in my [GitHub repository][4]. Therefore, to be able to use this, you need to fork my repository and [add it as an external repository source in your Azure DevTest lab][5].

![](/images/adtl1.png)

Once this custom repository is added, here is how you use the PowerShell Core artifact.

Select the Virtual Machines blade in the Azure DevTest Labs.

![](/images/adtl2.png)

Click the VM, and then click on Artifacts and Apply Artifacts. In the search box, type PowerShell Core, and then click on the result in the Apply Artifacts blade.

![](/images/adtl3.png)

Click on the artifact and supply the parameters required.

![](/images/adtl4.png)

**Package URL** &#8211; The MSI URL for installing PowerShell Core. You can retrieve this from https://github.com/PowerShell/PowerShell/releases.

**Install C runtime for Windows OS prior to Windows Server 2016?** &#8211; Select True for Windows Server 2012 R2 or select False. This is needed for Windows Server 2012 R2 if you want to use WinRM for PowerShell remoting.

Click Add. This artifact installation might take a few minutes and once complete, you can access the install script verbose logs.

![](/images/adtl5.png)

This is it. You can, of course, install these artifacts (Windows or Linux) at the time of Azure DTL VM provisioning itself. And, you can do this deployment via an ARM template as well.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "newVMName": {
            "type": "string",
            "defaultValue": "PSCore1601"
        },
        "labName": {
            "type": "string",
            "defaultValue": "PSMagDTL"
        },
        "size": {
            "type": "string",
            "defaultValue": "Standard_A6"
        },
        "userName": {
            "type": "string",
            "defaultValue": "ravikanth"
        },
        "password": {
            "type": "securestring"
        },
        "PowerShell_Core_a.k.a_PowerShell_6_packageUrl": {
            "type": "string",
            "defaultValue": "https://github.com/PowerShell/PowerShell/releases/download/v6.0.0/PowerShell-6.0.0-win-x64.msi"
        },
        "PowerShell_Core_a.k.a_PowerShell_6_installCRuntime": {
            "type": "bool",
            "defaultValue": false
        },
        "labVirtualNetworkName" : {
            "type": "string"
        },
        "labSubnetName" : {
            "type" : "string"        
        }        
    },
    "variables": {
        "labVirtualNetworkId": "[resourceId('Microsoft.DevTestLab/labs/virtualnetworks', parameters('labName'), parameters('labVirtualNetworkName'))]",
        "vmId": "[resourceId ('Microsoft.DevTestLab/labs/virtualmachines', parameters('labName'), parameters('newVMName'))]",
        "vmName": "[concat(parameters('labName'), '/', parameters('newVMName'))]"
    },
    "resources": [
        {
            "apiVersion": "2017-04-26-preview",
            "type": "Microsoft.DevTestLab/labs/virtualmachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "properties": {
                "labVirtualNetworkId": "[variables('labVirtualNetworkId')]",
                "notes": "Windows Server 2016 Datacenter",
                "galleryImageReference": {
                    "offer": "WindowsServer",
                    "publisher": "MicrosoftWindowsServer",
                    "sku": "2016-Datacenter",
                    "osType": "Windows",
                    "version": "latest"
                },
                "size": "[parameters('size')]",
                "userName": "[parameters('userName')]",
                "password": "[parameters('password')]",
                "isAuthenticationWithSshKey": false,
                "artifacts": [
                    {
                        "artifactId": "[resourceId('Microsoft.DevTestLab/labs/artifactSources/artifacts', parameters('labName'), 'privaterepo596', 'windows-powershellcore')]",
                        "parameters": [
                            {
                                "name": "packageUrl",
                                "value": "[parameters('PowerShell_Core_a.k.a_PowerShell_6_packageUrl')]"
                            },
                            {
                                "name": "installCRuntime",
                                "value": "[parameters('PowerShell_Core_a.k.a_PowerShell_6_installCRuntime')]"
                            }
                        ]
                    }
                ],
                "labSubnetName": "[parameters('labSubnetName')]",
                "disallowPublicIpAddress": true,
                "storageType": "Standard",
                "allowClaim": false,
                "networkInterface": {
                    "sharedPublicIpAddressConfiguration": {
                        "inboundNatRules": [
                            {
                                "transportProtocol": "tcp",
                                "backendPort": 3389
                            }
                        ]
                    }
                }
            }
        }
    ],
    "outputs": {
        "labVMId": {
            "type": "string",
            "value": "[variables('vmId')]"
        }
    }
}
```


At the end of this template deployment, you will have a Windows Server 2016 VM with PowerShell Core installed.

[1]: https://github.com/powershell/powershell
[2]: https://blogs.msdn.microsoft.com/powershell/2018/01/10/powershell-core-6-0-generally-available-ga-and-supported/
[3]: http://www.powershellmagazine.com/2016/08/22/azure-devtest-labs-artifact-for-installing-powershell-on-linux/
[4]: https://github.com/rchaganti/azure-devtestlab
[5]: https://docs.microsoft.com/en-us/azure/devtest-lab/devtest-lab-add-artifact-repo