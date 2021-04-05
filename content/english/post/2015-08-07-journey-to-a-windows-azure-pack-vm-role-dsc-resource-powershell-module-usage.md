---
title: 'Journey to a Windows Azure Pack VM Role DSC Resource: PowerShell Module Usage'
author: Ben Gelens
type: post
date: 2015-08-07T16:00:20+00:00
url: /2015/08/07/journey-to-a-windows-azure-pack-vm-role-dsc-resource-powershell-module-usage/
views:
  - 20826
post_views_count:
  - 6458
categories:
  - Azure Pack
  - PowerShell DSC
tags:
  - Azure Pack
  - PowerShell DSC

---
In this series,

[Part 1 - Journey to a Windows Azure Pack VM Role DSC Resource: Introduction](/2015/07/28/journey-to-a-windows-azure-pack-vm-role-dsc-resource/)

Part 2 - Journey to a Windows Azure Pack VM Role DSC Resource: PowerShell module usage (this article)

[Part 3 - Journey to a Windows Azure Pack VM Role DSC Resource: Inside the module](/2015/09/01/journey-to-a-windows-azure-pack-vm-role-dsc-resource-inside-the-module/)

[Part 4 - Journey to a Windows Azure Pack VM Role DSC Resource: The DSC resource](/2015/09/22/part-4-journey-to-a-windows-azure-pack-vm-role-dsc-resource-the-dsc-resource/)


In the [previous post][1] you have seen that there is no DSC resource yet to deploy Windows Azure Pack VM Roles with. Also you have seen that using the Azure PowerShell module still leaves some blank spots which makes it a bit difficult to use. The dependency on the publish settings file is another factor which make the use of Azure PowerShell module not the ideal solution for every scenario.

In this article we take a look at VM Role deployment using my PowerShell module which should be making all this a bit easier.

### Windows Azure Pack Tenant and Tenant Public API

Windows Azure Pack has 2 API’s dealing with tenant operations.

  * The Tenant API is considered a highly privileged API and should not be exposed to the public internet.
  * The Tenant Public API. As the name implies, the API to expose publicly.

The Tenant API by default listens on port 30005 and has a self-signed certificate assigned. The Tenant Public API by default listens on port 30006 and has a self-signed certificate assigned as well. Since the nature of the Tenant Public API is to serve publicly, it will most likely be reconfigured to listen on port 443 and have a commercially signed certificate bound. The PowerShell module has to be able to deal with these scenarios and based on user choice ignore errors with certificates or not.

### Prerequisites

For the module to work against the Windows Azure Pack Tenant Public API we need to configure the API to accept bearer tokens together with certificates. By default, the API is configured in what is known as “Native” mode. We need to configure it in Hybrid mode to work with the bearer tokens.

Run the following code on all the servers hosting the Tenant Public API:

```powershell
Unprotect-MgmtSvcConfiguration -Namespace tenantpublicapi
Set-WebConfigurationProperty -PSPath IIS:\Sites\MgmtSvc-TenantPublicAPI -Filter "appSettings/add[@key='TenantServiceMode']" -Name 'value' -Value 'HybridTenant'
Protect-MgmtSvcConfiguration -Namespace tenantpublicapi
```


The change is effective immediately, no need to restart IIS or anything.

The Tenant API works out of the box with bearer tokens, so no need to change configurations.

From my experience, some permissions are missing in the Azure Pack Store database for the Tenant Public API to function correctly under Hybrid mode. Specifically means to check on invalidated user tokens. To fix this, run the following TSQL code against the database:

```powershell
USE [Microsoft.MgmtSvc.Store]
GO
Grant Execute On Type::.mp.CoAdminTableType To mp_TenantAPI
Grant Execute On Object::mp.GetInvalidatedUserTokens To mp_TenantAPI
```

### The PowerShell module

As I wrote in the previous article, the module discussed here can be downloaded from [GitHub](https://github.com/bgelens/WAPTenantPublicAPI ) or from the PowerShell gallery, if you have WMF5 installed, by running _Install-Module WAPTenantPublicAPI_.

### Example deployment

This article focuses on getting you familiar with the module. In the next article I will highlight some code used in this module.  First have a look at an example script which uses the functions from this module and go from there.

```powershell
Get-WAPToken -Url https://sts.bgelens.nl -ADFS -Credential administrator@gelens.int
Connect-WAPAPI -Url https://api.bgelens.nl -Port 443
Get-WAPSubscription -Name Test | Select-WAPSubscription
$GI = Get-WAPGalleryVMRole -Name DSCPullServerClient
$OSDisk = $GI | Get-WAPVMRoleOSDisk | Sort-Object -Property AddedTime -Descending | Select-Object -First 1
$NW = Get-WAPVMNetwork -Name internal
$VMProps = New-WAPVMRoleParameterObject -VMRole $GI -OSDisk $OSDisk -VMRoleVMSize Medium -VMNetwork $NW
$VMProps.VMRoleAdminCredential = 'Administrator:Welkom01'
$VMProps.DSCPullServerClientCredential = 'Domain\Certreq:password'
$VMProps.DSCPullServerClientConfigurationId = '7844f909-1f2e-4770-9c97-7a2e2e5677ae'
New-WAPVMRoleDeployment -VMRole $GI -ParameterObject $VMProps -CloudServiceName MyCloudService
Get-WAPCloudService -Name MyCloudService | Get-WAPVMRole
```


First you acquire the token by authenticating against either the Windows Azure Pack authentication website or ADFS (use the –ADFS switch in this case) using the _Get-WAPToken_ function.

![](/images/wapdsc1.png)

This function will populate some variables (Token and Headers) defined in the module level so other functions can now access the data. The Headers variable will contain a hash table once successfully authenticated.

```powershell
@{
    Authorization = "Bearer $Token"
    'x-ms-principal-id' = $Credential.UserName
    Accept = 'application/json'
}
```


These headers are used with each subsequent function which target the API. It tells the API:

  * To handle Bearer token authentication and supplies the JWT token as a value.
  
    The API will check the claims and other validating data from the token.
  * The principal identifier (who you are)
  
    It is the same as the credential used to acquire the JWT token (<someone@somedomain.tld>).
  * The client (invoker) understands JSON and wants to retrieve the potential return data as JSON.
  
    When the API is not told the client is capable to deal with JSON, a lot less data is returned.

Next we “connect” with Windows Azure Pack using the _Connect-WAPAPI_ function.

![](/images/wapdsc2.png)

This function actually just checks if we can successfully access the subscription URL using the headers and bearer token. When this is the case, some module-scoped variables will be populated with data (PublicTenantAPIUrl, Port, and IgnoreSSL). Subsequent functions will check if these variables are $null or have data before executing the code intended. If $null, an exception will be thrown telling you to first “connect”.

![](/images/wapdsc3.png)

Just like the Azure PowerShell module, the subscription is the basis for operations in this module. We get a specific subscription _(Get-WAPSubscription)_ and make it the current one by piping it to _Select-WAPSubscription._ This will populate the last module-scoped variable, Subscription.  Now everything is in place to interact with the API.

![](/images/wapdsc4.png)

A subscription is bound to a plan. VM Roles come as gallery items and are assigned to a subscription via the plan or via add-ons. To know which VM Roles are available to us, we need to interrogate the API. We do this with the _Get-WAPGalleryVMRole_ function. Once we know which one we want to deploy, we specifically look it up by specifying the name and, if multiple versions are available, by version. The resulting object is stored in a variable as it contains most of the info we need to generate the required JSON for the resource definition.

![](/images/wapdsc5.png)

The tags used to lookup the Operating System disks are part of the view definition which was acquired by getting the VM Role gallery item.

![](/images/wapdsc6.png)

Therefore, the gallery item object is passed to the _Get-WAPVMRoleOSDisk_ function. On the right hand side of the pipeline we sort the resulting objects based on their AddedTime property and select the first one effectively selecting the latest one. The output is captured in a variable as it contains some info we need to generate the JSON for the resource definition later on.

![](/images/wapdsc7.png)

Next we find out what networks are available for our VM Role deployment using the _Get-WAPVMNetwork_ function. VM Networks are assigned to a subscription on the plan or add-on level by the Azure Pack Admin. Besides this, a tenant is able to create VM Networks themselves. The output is again stored in a variable as we need it in the next phase.

![](/images/wapdsc8.png)

Most of what you have seen until now can all be achieved by using the Azure module itself. The biggest difference being the VM Role gallery item with its properties. The API provides you all that you need to construct the correct JSON for the resource definition but the Azure module apparently was not designed to handle this. Now let’s start creating that JSON. The _New-WAPVMRoleParameterObject_ function handles what would normally be done by the view definition wizard in the tenant portal.

![](/images/wapdsc9.png)

First look at the help for the function.

![](/images/wapdsc10.png)

As you can see this function has the -Interactive switch parameter. When you deal with a VM Role for the first time, you probably want to run it in the Interactive mode as it will prompt you to fill in certain types of values and provide you with default selections for option parameters. To better show this, look at the screenshot below.

![](/images/wapdsc11.png)

You provide the function with all objects it requires which were gathered before:

  * VM Role Gallery Item object
  * VM Role compatible OSDisk
  * VM Network available on the subscription

Also specified is the VM Role VM Size from which the value is taken from a validation set.

Then, because we specified the -Interactive switch, the function starts to prompt you for missing information. Properties which require credentials for example will ask you to enter credentials in a certain format. The input is checked for proper formatting (domain\Username:Password or Username:Password) and if it would be invalid, the user would be prompted again.

To give you a visual reference, in the tenant portal it would like this:

![](/images/wapdsc12.png)

All parameters which have defaults defined will not prompt the user but instead, go with the defaults. Properties which do not have defaults configured will prompt the user for input.

If properties are of the “option” type, the default value will be offered as default and the option values will be enumerated so the user can type in a correct value (which is checked to be valid).

To give you a visual reference, in the tenant portal a property of the type option would like this:

![](/images/wapdsc13.png)

(Note: Because the VM Role Gallery Item is very flexible in the way it can be constructed; this function probably does not deal with every possible scenario. If you find your VM Role can’t be deployed with this module, the cause probably lies in the _New-WAPVMRoleParameterObject_ function. If this is that case, please log a detailed incident on [GitHub][2] or fork the project and do a pull request of the modified code.)

Now if we run this function without the -Interactive switch you will see we have a few blanks.

![](/images/wapdsc14.png)

These are left with a $null value as we have no way to know what should be in them up front. The values have to be provided after the object is generated. This mode is of course the mode you want to use when you know what the VM Role properties need to be and deploy the VM Role using a script, SMA runbook or through the use of a DSC resource for example.

![](/images/wapdsc15.png)

Now we have the parameter object we can deploy the VM Role using the function _New-WAPVMRoleDeployment._ This function requires you to pass the VM Role gallery item object, the parameter object which was just generated and enriched, and finally a cloud service name. The function will construct the JSON payload and sends it to the API.

I’ve designed the function to behave similar to the Windows Azure Pack tenant portal which means, a cloud service can only contain one VM Role (which could contain multiple VM instances) and it has the same name as the cloud service. Technically we could provision multiple VM Roles to the same cloud service but I feel because the tenant portal RP was not designed to handle this, we should avoid this.

![](/images/wapdsc16.png)

Finally, we can monitor for the deployment to finish using _Get-WAPCloudService._

![](/images/wapdsc17.png)

And if we want, we can remove the deployment by running _Get-WAPCloudService_ and piping the result to _Remove-WAPCloudService._

In the next article, I will show some of the inner workings of the module so you have a better understanding of what is going on.
