---
title: 'Part 4 – Journey to a Windows Azure Pack VM Role DSC Resource: The DSC resource'
author: Ben Gelens
type: post
date: 2015-09-22T16:58:15+00:00
url: /2015/09/22/part-4-journey-to-a-windows-azure-pack-vm-role-dsc-resource-the-dsc-resource/
views:
  - 8600
post_views_count:
  - 1327
categories:
  - PowerShell DSC
  - Azure Pack
tags:
  - Azure Pack
  - PowerShell DSC

---
In this series:

[Part 1 &#8211; Journey to a Windows Azure Pack VM Role DSC Resource: Introduction][1]

[Part 2 &#8211; Journey to a Windows Azure Pack VM Role DSC Resource: PowerShell module usage][2]

[Part 3 &#8211; Journey to a Windows Azure Pack VM Role DSC Resource: Inside the module][3]

Part 4 &#8211; Journey to a Windows Azure Pack VM Role DSC Resource: The DSC resource (this article)

You’ve reached the final article in this series. Now the ground work has been done by creating a PowerShell module, it’s time to layer a DSC resource on top of it. While reading this post, you will see that the investment needed to do this is not such a big deal compared to the creation of the PowerShell module. When you are developing modules I encourage you to take the extra step!

As with the [PowerShell module][4], I have made the DSC resource module explained in this post available on [GitHub][5]. If you have WMF 5.0 installed, you can install the DSC resource module from the PowerShell gallery as well by running the following command: **Install-Module cWAPack**. For the PowerShell module run **Install-Module WAPTenantPublicAPI**.

### Background

Some time ago I’ve written a blog series on [Integrating Windows Azure Pack VM Roles with DSC Pull service][6]. I’m going to use the resulting VM Role to demonstrate deployment of VM Roles using DSC where the resulting VM will be configured using DSC as well (the “Inception” effect). Also, I gave a presentation on the [Dutch PowerShell User Group][7], where the deployment of the VM Role was partly backed by Service Management Automation. I did this so we could have one generic DSC Pull-enabled VM Role which could become any kind of service by making deployment time decisions on the SMA side of things (e.g. install sources to attach, extra disks to attach, keep VM Role under provisioning status while DSC was still running). I refer to this process, jokingly, as “Just in time automation”. You can read about it [here][8].  Finally, I would like to mention a blog post I’ve written on a way to “[Bring your own DSC][9]” as a way to implement the DSC VM Extension behavior with the somewhat limited Azure Pack VM Role capabilities.

With this blog series, I feel like I’ve come full circle now by creating an end-2-end DSC-enabled Azure Pack VM Role deployment model. Hope you enjoy reading about this as much as I had learning it all!

### Creating the DSC resource

#### Class- or Script-based resource?

I’m a big fan of authoring class-defined DSC resources. I think the coding experience is far superior than script-based resource modules and most of the complexities involved with a script-based resource module are non-existent for class-based resource modules (e.g. folder structure, MOF schema, etc.). One big advantage of the script-based DSC resource modules however is compatibility with WMF 4.0 (DSC v1) and since WMF 5.0 (although in Production Preview) is still not RTM at the time of writing, I’ve decided to develop this resource module as a script-based module. Another reason to pick script-based resource module development over class-based, or vice versa, could be target technology. If a technology is for Windows Server 2016+ only for example (e.g. [containers][10]), I would always choose class-based development.

#### xDSCResourceDesigner

To help you set up a DSC script module correctly (using the correct folder structure, MOF schema file, etc.) Microsoft released a PowerShell helper module which can handle this for you. The module is called xDSCResourceDesigner and can be found on [GitHub][11] and the [PowerShell gallery][12] (**Install-Module xDSCResourceDesigner**).

Once this module is installed we can create a DSC resource script module using a PowerShell script. I used the following to create mine:

```powershell
Import-Module xDSCResourceDesigner
$BuildDir = 'C:\'

New-xDscResource -ModuleName cWAPack -Name 'BG_WAPackVMRole' -FriendlyName 'WAPackVMRole' -ClassVersion '0.0.3.0' -Path $BuildDir -Property @(
    New-xDscResourceProperty -Name Name -Type String -Attribute Key -Description 'Cloud Service and VM Role Name'
    New-xDscResourceProperty -Name Ensure -Type String -Attribute Write -ValidateSet 'Present','Absent'
    New-xDscResourceProperty -Name Url -Type String -Attribute Required -Description 'Tenant Public API or Tenant API URL'
    New-xDscResourceProperty -Name SubscriptionId -Type String -Attribute Required -Description 'Subscription ID'
    New-xDscResourceProperty -Name Credential -Type PSCredential -Attribute Required -Description 'Credentials to acquire token'
    New-xDscResourceProperty -Name VMRoleGIName -Type String -Attribute Required -Description 'VM Role Gallery Item name'
    New-xDscResourceProperty -Name VMRoleGIVersion -Type String -Attribute Write -Description 'VM Role Gallery Item Version. Specify if multiple versions are published'
    New-xDscResourceProperty -Name VMRoleGIPublisher -Type String -Attribute Write -Description 'VM Role Gallery Item Publisher. Specify if multiple VM Roles with the same name but different publishers are published'
    New-xDscResourceProperty -Name VMSize -Type String -Attribute Write -ValidateSet 'Small','A7','ExtraSmall','Large','A6','Medium','ExtraLarge'
    New-xDscResourceProperty -Name OSDiskSearch -Type String -Attribute Write -ValidateSet 'LatestApplicable','LatestApplicableWithFamilyName','Specified'
    New-xDscResourceProperty -Name OSDiskFamilyName -Type String -Attribute Write
    New-xDscResourceProperty -Name OSDiskRelease -Type String -Attribute Write
    New-xDscResourceProperty -Name NetworkReference -Type String -Attribute Required
    New-xDscResourceProperty -Name VMRoleParameters -Type Hashtable -Attribute Write
    New-xDscResourceProperty -Name TokenSource -Type String -Attribute Required -ValidateSet 'ASPNET','ADFS'
    New-xDscResourceProperty -Name TokenUrl -Type String -Attribute Required
    New-xDscResourceProperty -Name TokenPort -Type Uint16 -Attribute Write -Description 'Specify custom port to acquire token. Defaults for ADFS: 443, ASP.Net: 30071'
    New-xDscResourceProperty -Name Port -Type Uint16 -Attribute Write -Description 'Specify API port. Default: 30006'
) -Force
```

Running this script will create a manifest module called cWAPack. Inside the module, a DSC resource BG_WAPackVMRole with a friendly name of WAPackVMRole is created. The schema.mof file for this resource is generated as well.

![](/images/wap1.png)

You can read more on the file structure and schema.mof file content here: <https://technet.microsoft.com/en-us/library/dn956964.aspx>

Including in the creation script, I defined what parameters will be used by the WAPackVMRole resource. These parameters will end up being declared in the schema.mof file and the resource script module file.

How do you come up with the parameters for the DSC resource? Just create an end-2-end deployment using the PowerShell module. Capture the steps in an “orchestration” script and figure out what the variables are to make the script generically usable. Those variables will be the parameters. If you don’t have them all clear from the get go, you can always adjust the schema.mof and script module files later to include them. Also, the xDSCResourceDesigner module has a function Update-xDscResource which can handle this for you.

The resource module script generated by the designer will have the 3 DSC functions, Get-TargetResource, ,Set-TargetResource, and Test-TargetResource, included.

![](/images/wap2.png)

These functions will have the parameters defined in the creation script included already. Note for now that the Get-TargetResource will only have the mandatory parameters included. Test and Set have all parameters included.

#### Visual Studio Project

PowerShell developing in Visual Studio has become my own personal preference. You can of course use the tooling of your own choosing.

I like VS as it has everything in house I’m currently looking for. E.g. JSON syntax/schema support, PowerShell support (by the awesome [PoshTools][13]!) including rich debugging, Solution/Project structure, Git /GitHub support, Azure SDK, and so on.

Since I created the DSC resource module outside of VS, I’m going to create a solution/project from the directory. Within VS I go to the File menu and choose, New Project and select the Script Project (no need to specify the module project as the module structure and manifest are already created).

![](/images/wap3.png)

Make sure to specify the parent of the DSC module you created as the location. Deselect Create directory for solution and provide the name of the DSC module (cWAPack). Then hit OK.

The new solution will be opened and a new item called script.ps1 is created and made part of the project. Remove it.

![](/images/wap4.png)

Next hit the “Show all files” button.

![](/images/wap5.png)

Right click the cWAPack.psd1 file and select “Include in project”. Do the same for the DSCResources folder. The items which have become part of the project are colorized for visual reference.

![](/images/wap6.png)

Now we have created a Visual Studio solution/project from our module, we can save the solution.

### Nested Module

Because we are creating a DSC resource which is dependent on a PowerShell module, we want to make sure the correct module with the correct version is always available to it. We currently could do that in a couple of ways:

  * Provide installation instructions to the user so he/she is made aware of the dependency and is responsible to fulfill the requirements.
  * Nest the Module with the DSC resource making sure it is always available to the DSC resource and the module used is of the correct version.

In this case I’m going to nest the module with the DSC resource module. To do this, copy the WAPTenantPublicAPI PowerShell module into the root of the cWAPack DSC resource module.

![](/images/wap7.png)

Open the visual studio solution and hit the “show all items” button. Right click the WAPTenantPublicAPI folder and select “include in project”.

![](/images/wap8.png)

Now the module is included in the DSC resource module directory, we need to modify the DSC resource module manifest. We need to do this so we are assured when the DSC resource module is loaded by the Local Configuration Manager, the WAPTenantPublicAPI PowerShell module which is included with the DSC resource module is loaded as well. Open the cWAPack.psd1 file. Navigate to the NestedModules Key and uncomment it. Within the array, type WAPTenantPublicAPI and save the manifest.

![](/images/wap9.png)

I’ve debugged the LCM while a configuration was processed. This showed me that the nested module is loaded even though the same PowerShell module exists in the system in non-nested form. Nesting this makes sure that the PowerShell module on which the DSC resource module is developed on and relies on is always used.

### Coding it up

Now it’s time to code up the BG\_WAPackVMRole resource. Open it up by navigating in the solution project to DSCResources\BG\_WAPackVMROle and double click BG_WAPackVMRole.psm1 which will open the file for editing.

First navigate to the Set-TargetResource function and copy the entire param block. Then overwrite the Get-TargetResource param block by removing its current param block and pasting in the param block from the clipboard. I do this so we can be sure the Get-DscConfiguration cmdlet is able to return all user specified information used for provisioning without the need to query it all out interactively (making Get less involved/heavy). As stated earlier, if you leave it as default, the Get-TargetResource will only have the mandatory parameters assigned.

Authentication and subscription selection is something every function needs to do. It therefore makes sense to create a little helper function to handle these tasks so we don’t end up with a lot of redundant code. I created a helper function called setup to handle this.


```powershell
function Setup {
    param (
        $TokenSource,
        $TokenUrl,
        $TokenPort,
        $Credential,
        $Url,
        $Port,
        $SubscriptionId
    )
    try {
        if ($TokenSource -eq 'ADFS') {
            Write-Verbose "Acquiring ADFS token from $TokenUrl with credentials: $($Credential.username)"
            Get-WAPToken -Credential $Credential -URL $TokenUrl -Port $TokenPort -ADFS
        } else {
            Write-Verbose "Acquiring ASP.Net token from $TokenUrl"
            Get-WAPToken -Credential $Credential -URL $TokenUrl -Port $TokenPort
        }
        Connect-WAPAPI -URL $Url -Port $Port    
    $Subscription = Get-WAPSubscription -Id $SubscriptionId
    if ($null -eq $SubscriptionId) {
        throw "Subscription with Id: $SubscriptionId was not found!"
    }
    $Subscription | Select-WAPSubscription
} catch { 
    Write-Error -ErrorRecord $_ -ErrorAction Stop
}
}
```
The function has a bunch of parameters. Note that none of the parameters have the mandatory argument. I don’t assign mandatory because as once as you define a parameter as mandatory, you cannot splat a hash table which contains more information then defined in the param block against it anymore (not without specifying a parameter with ValueFromRemainingArguments argument. You can read more about this [here][14].).

The function makes use of the WAPTenantPublicAPI module to acquire a JWT token to interact with the Azure Pack API and selecting the subscription to work against. As you can see, this is basically the start of any “orchestration” script to deploy VM Roles with.

Next we look at the Test-TargetResource function.


```powershell
function Test-TargetResource {
    [CmdletBinding()]
    [OutputType([System.Boolean])]
    param (
        [parameter(Mandatory)]
        [String] $Name,    
    	[ValidateSet('Present','Absent')]
    	[String] $Ensure,

   		[parameter(Mandatory)]
    	[String] $Url,
    	[parameter(Mandatory)]
    	[String] $SubscriptionId,

    	[parameter(Mandatory)]
    	[PSCredential] $Credential,

    	[parameter(Mandatory)]
    	[String] $VMRoleGIName,

    	[String] $VMRoleGIVersion,

    	[String] $VMRoleGIPublisher,

    	[ValidateSet('Small','A7','ExtraSmall','Large','A6','Medium','ExtraLarge')]
    [String] $VMSize = 'Medium',

    [ValidateSet('LatestApplicable','LatestApplicableWithFamilyName','Specified')]
    [String] $OSDiskSearch = 'LatestApplicable',

    [String] $OSDiskFamilyName,

    [String] $OSDiskRelease,

    [parameter(Mandatory)]
    [String] $NetworkReference,

    [Microsoft.Management.Infrastructure.CimInstance[]] $VMRoleParameters,

    [parameter(Mandatory)]
    [ValidateSet('ASPNET','ADFS')]
    [String] $TokenSource,

    [parameter(Mandatory)]
    [String] $TokenUrl,

    [UInt16] $TokenPort,

    [UInt16] $Port
)
try {
    Setup @PSBoundParameters

    $VMRole = Get-WAPVMRole -CloudServiceName $Name -ErrorAction SilentlyContinue

    if ($Ensure -eq 'Present') {
        if ($null -ne $VMRole) {
        return $true
        } else {
            return $false
        }
    } else {
        if ($null -eq $VMRole) {
            return $true
        } else {
            return $false
        }
    }
} catch {
    Write-Error -ErrorRecord $_ -ErrorAction Stop
}
}
```
This function is really simple. It starts by splatting the PSBoundParameters against the “Setup” helper function so we have a JWT token and a subscription selected within the current runspace. Then it queries the Azure Pack (Public) Tenant API if a VM Role exists by querying for a VMRole with the specified name. The ErrorAction for this function is defined as SilentlyContinue, this way the VMRole variable will either end up with a VM Role object or with Null. And then, based on the result, either true or false is returned respective to the ensure value. We could implement a complex testing algorithm but I figure this resource would only be used for deployment purposes and not for ensuring idempotency as the resource deployed will have its own lifecycle which will have nothing to do with the DSC configuration it was deployed with. So I keep it simple!

Next we look at the Set-TargetResource function.


```powershell
function Set-TargetResource {
    [CmdletBinding()]
    param (
        [parameter(Mandatory)]
        [String] $Name,   
    [ValidateSet('Present','Absent')]
    [String] $Ensure,

    [parameter(Mandatory)]
    [String] $Url,

    [parameter(Mandatory)]
    [String] $SubscriptionId,

    [parameter(Mandatory)]
    [PSCredential] $Credential,

    [parameter(Mandatory)]
    [String] $VMRoleGIName,

    [String] $VMRoleGIVersion,

    [String] $VMRoleGIPublisher,

    [ValidateSet('Small','A7','ExtraSmall','Large','A6','Medium','ExtraLarge')]
    [String] $VMSize = 'Medium',

    [ValidateSet('LatestApplicable','LatestApplicableWithFamilyName','Specified')]
    [String] $OSDiskSearch = 'LatestApplicable',

    [String] $OSDiskFamilyName,

    [String] $OSDiskRelease,

    [parameter(Mandatory)]
    [String] $NetworkReference,

    [Microsoft.Management.Infrastructure.CimInstance[]] $VMRoleParameters,

    [parameter(Mandatory)]
    [ValidateSet('ASPNET','ADFS')]
    [String] $TokenSource,

    [parameter(Mandatory)]
    [String] $TokenUrl,
 
    #do not define default as functions for ADFS and ASP have different defaults
    [UInt16] $TokenPort,

    [UInt16] $Port = 30006
)

try {
    Setup @PSBoundParameters

    if ($Ensure -eq 'Absent') {
        Get-WAPCloudService -Name $Name | Remove-WAPCloudService -Force | Out-Null
    } else {
        #Get GI with Name
        $GI = Get-WAPGalleryVMRole -Name $VMRoleGIName
        #If Multiple GI's returned, check if user specified version and select on that
        if ($GI -is [array] -and $VMRoleGIVersion) {
            $GI = Get-WAPGalleryVMRole -Name $VMRoleGIName -Version $VMRoleGIVersion
        }
        #If Multiple GI's returned, and version not specified or also returned multiple object, check if user specified Publisher and select on that.
        if ($GI -is [array] -and $VMRoleGIPublisher) {
            $GI = $GI | Where-Object -FilterScript {$_.Publisher -eq $VMRoleGIPublisher}
        }
        #If No GI is left, throw error.
        if ($null -eq $GI) {
            throw 'No VM Role Gallery Item found matching user criteria'
        } else {
            $GI | Out-String | Write-Verbose
        }

        if ($OSDiskSearch -eq 'LatestApplicable') {
            $OSDisk = $GI | Get-WAPVMRoleOSDisk | 
                Sort-Object Addedtime -Descending | 
                    Select-Object -First 1
        } elseif ($OSDiskSearch -eq 'LatestApplicableWithFamilyName') {
            $OSDisk = $GI | Get-WAPVMRoleOSDisk | 
                Where-Object -FilterScript {$_.FamilyName -eq $OSDiskFamilyName} | 
                    Sort-Object Addedtime -Descending | Select-Object -First 1
        } elseif ($OSDiskSearch -eq 'Specified') {
            $OSDisk = $GI | Get-WAPVMRoleOSDisk | 
                Where-Object -FilterScript {$_.FamilyName -eq $OSDiskFamilyName -and $_.Release -eq $OSDiskRelease}
        }
        if ($null -eq $OSDisk) {
            throw 'No valid OS disk was found matching User provided criteria'
        }
        $OSDisk | Out-String | Write-Verbose

        $Net = Get-WAPVMNetwork -Name $NetworkReference
        if ($null -eq $Net) {
            throw 'No valid virtual network was found'
        }
        $net | Out-String | Write-Verbose

        $VMProps = New-WAPVMRoleParameterObject -VMRole $GI -OSDisk $OSDisk -VMRoleVMSize $VMSize -VMNetwork $Net
        foreach ($P in $VMRoleParameters) {
            Add-Member -InputObject $VMProps -MemberType NoteProperty -Name $P.key -Value $P.value -Force
        }
        $VMProps | Out-String | Write-Verbose

        New-WAPVMRoleDeployment -VMRole $GI -ParameterObject $VMProps -CloudServiceName $Name | Out-Null
    }
} catch {
    Write-Error -ErrorRecord $_ -ErrorAction Stop
}
}
```
This function is a bit more involved. It also starts with the Setup helper function (remember that each function will run in its own runspace when invoked by the LCM, so we don’t share the environment between a Test and a Set). Next, based on the Ensure parameter, it will either start a provisioning or a deletion respectively.  In the case of provisioning, first it will get the Gallery Item on which to base the deployment off. Then it will search for a compatible OS disk (containing the correct Tags). The VM Network will be looked up and then the VMRole parameter object will be generated. Based on user specified hash table in the DSC configuration, this object will be enriched with the required data (we will look at an example deployment a bit later). Finally, the deployment is started.

Finally, we look at the Get-TargetResource function.


```powershell
function Get-TargetResource {
    [CmdletBinding()]
    [OutputType([System.Collections.Hashtable])]
    param (
        [parameter(Mandatory)]
        [String] $Name,    
    [ValidateSet('Present','Absent')]
    [String] $Ensure,

    [parameter(Mandatory)]
    [String] $Url,

    [parameter(Mandatory)]
    [String] $SubscriptionId,

    [parameter(Mandatory)]
    [PSCredential] $Credential,

    [parameter(Mandatory)]
    [String] $VMRoleGIName,

    [String] $VMRoleGIVersion,

    [String] $VMRoleGIPublisher,

    [ValidateSet('Small','A7','ExtraSmall','Large','A6','Medium','ExtraLarge')]
    [String] $VMSize = 'Medium',

    [ValidateSet('LatestApplicable','LatestApplicableWithFamilyName','Specified')]
    [String] $OSDiskSearch = 'LatestApplicable',

    [String] $OSDiskFamilyName,

    [String] $OSDiskRelease,

    [parameter(Mandatory)]
    [String] $NetworkReference,

    [Microsoft.Management.Infrastructure.CimInstance[]] $VMRoleParameters,

    [parameter(Mandatory)]
    [ValidateSet('ASPNET','ADFS')]
    [String] $TokenSource,

    [parameter(Mandatory)]
    [String] $TokenUrl,
 
    #do not define default as functions for ADFS and ASP have different defaults
    [UInt16] $TokenPort,

    [UInt16] $Port = 30006
)

Setup @PSBoundParameters

if (Get-WAPVMRole -CloudServiceName $Name -ErrorAction SilentlyContinue) {
    $Ensure = 'Present'
} else {
    $Ensure = 'Absent'
}
Add-Member -InputObject $PSBoundParameters -MemberType NoteProperty -Name 'Ensure' -Value $Ensure
$PSBoundParameters.Remove('Verbose')
$PSBoundParameters.Remove('Debug')
Write-Output -InputObject $PSBoundParameters
}
```
Again, the setup helper function is called first. Then based upon the VM Role being deployed or not, Ensure is set to Absent or Present and added to the PSBoundParameters. Potentially Verbose and Debug are removed from the PSBoundParameters hash table as this hash table is to be output as the Class object and these parameters are not explicitly defined for the DSC resource class (if we do not do this, the Get-DscConfiguration will error out as the object does not correspond to the class definition). Finally, we send the PSBoundParameters hashtable to the output stream.

There you have it! Not much work to add on a DSC resource on top of your PowerShell module if you ask me. Save it and put it in your modules directory. Ready to roll.

### Deploying a VM Role using DSC

The DSC resource module is finished. To make it available for the LCM we need to copy it over to ‘c:\Program Files\WindowsPowerShell\Modules’. Now let’s look at an example configuration script.




```powershell
configuration WAPVMRole {
    param (
        [PSCredential] $Credential
    )
    Import-DscResource -ModuleName cWAPack
    Import-DscResource -ModuleName PSDesiredStateConfiguration
node $AllNodes.NodeName {
    WAPackVMRole DSCClient {
        VMSize = 'Medium'
        Name = 'TestDSC'
        SubscriptionId = 'b5a9b263-066b-4a8f-87b4-1b7c90a5bcad'
        Url = 'https://api.bgelens.nl'
        Credential = $Credential
        VMRoleGIName = 'DSCPullServerClient'
        OSDiskSearch = 'LatestApplicable'
        NetworkReference = 'Internal'
        TokenSource = 'ADFS'
        TokenUrl = 'https://sts.bgelens.nl'
        TokenPort = 443
        Ensure = 'Present'
        Port = 443
        VMRoleParameters = @{
            VMRoleAdminCredential = 'Administrator:P@$Sw0rd!'
            DSCPullServerClientConfigurationId = '7844f909-1f2e-4770-9c97-7a2e2e5677ae'
            DSCPullServerClientCredential = 'Domain\certreq:Password!'
        }
    }
}
}
```
The VM Role parameters are being send to the DSC resource as a hash table. How do we know again what VMRoleParameters we need to pass? It’s all described in Part 2: [Module usage][2]. Basically, you need to do some investigation up front as you need to interrogate the API and the Gallery Item to know what values you can pass. This VM Role will configure the LCM in the “to be deployed” VM to be a pull client.

Now let’s call the configuration.

```powershell
$Cred = New-Object -TypeName pscredential -ArgumentLis @('ben@bgelens.nl', (ConvertTo-SecureString -String 'MySecurePWD!' -AsPlainText -Force))
$configdata = @{
    AllNodes = @(
        @{
            NodeName = 'localhost'
            PSDscAllowPlainTextPassword = $true
            PSDscAllowDomainUser = $true 
        } 
    )
}
WAPVMRole -ConfigurationData $configdata -Credential $Cred
Start-DscConfiguration .\WAPVMRole -Wait –Verbose
```


In this case I generate a Credential object to be used for interacting with the Public Tenant API. As I don’t have a certificate to encrypt the sensitive data within the MOF file with, I create a configuration data hash table and set PSDscAllowPlainTextPassword to True. In WMF 5.0 a new warning will be thrown when you use UPN or Domain\Username for credentials if the configuration targets localhost. To suppress this, I added PSDscAllowDomainUser to the configuration data hash table. Finally, I call the configuration so the MOF file is generated and call Start-DscConfiguration to make it so.

![](/images/wap10.png)

When viewed from the portal, we see the VM Role being deployed.

![](/images/wap11.png)

We can of course also use the PowerShell module to check on this:

![](/images/wap12.png)

There you have it! I hope you enjoyed this journey to a DSC resource series. If you are interested, keep watching my GitHub repo as I’m still actively developing both the PowerShell module and the DSC resource.

[1]: /2015/07/28/journey-to-a-windows-azure-pack-vm-role-dsc-resource/
[2]: /2015/08/07/journey-to-a-windows-azure-pack-vm-role-dsc-resource-powershell-module-usage/
[3]: /2015/09/01/journey-to-a-windows-azure-pack-vm-role-dsc-resource-inside-the-module/
[4]: https://github.com/bgelens/WAPTenantPublicAPI
[5]: https://github.com/bgelens/cWAPack
[6]: http://www.hyper-v.nu/archives/bgelens/2015/02/integrating-vm-role-with-desired-state-configuration-part-1-introduction-and-scenario/
[7]: http://www.dupsug.com/
[8]: http://www.hyper-v.nu/archives/bgelens/2015/06/lessons-learned-dsc-pull-security-and-integration-with-wapack-dupsug/
[9]: http://www.hyper-v.nu/archives/bgelens/2015/04/byo-dsc-with-vm-roles-a-vm-dsc-extension-alternative-for-wapack/
[10]: https://github.com/bgelens/cWindowsContainer
[11]: https://github.com/PowerShell/xDSCResourceDesigner/tree/master
[12]: https://www.powershellgallery.com/packages/xDSCResourceDesigner/
[13]: https://visualstudiogallery.msdn.microsoft.com/c9eb3ba8-0c59-4944-9a62-6eee37294597
[14]: https://technet.microsoft.com/en-us/library/hh847743.aspx