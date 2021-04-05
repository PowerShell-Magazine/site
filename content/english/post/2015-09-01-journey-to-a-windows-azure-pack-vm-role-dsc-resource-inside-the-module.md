---
title: 'Journey to a Windows Azure Pack VM Role DSC Resource: Inside the module'
author: Ben Gelens
type: post
date: 2015-09-01T16:00:58+00:00
url: /2015/09/01/journey-to-a-windows-azure-pack-vm-role-dsc-resource-inside-the-module/
views:
  - 18021
post_views_count:
  - 7975
categories:
  - Azure Pack
  - PowerShell DSC
tags:
  - Azure Pack
  - PowerShell DSC

---
In this series,

[Part 1 – Journey to a Windows Azure Pack VM Role DSC Resource: Introduction][1]

[Part 2 – Journey to a Windows Azure Pack VM Role DSC Resource: PowerShell module usage][2]

Part 3 – Journey to a Windows Azure Pack VM Role DSC Resource: Inside the module (this article)

[Part 4 – Journey to a Windows Azure Pack VM Role DSC Resource: The DSC resource][3]

In the [previous post][2] you have seen how to use the PowerShell module to deploy VM Roles via the Tenant API or Tenant Public API.

In this article we take a look at some of the module’s code so you have a better understanding of what is going on under the covers.

The module discussed here can be downloaded from <a href="https://github.com/bgelens/WAPTenantPublicAPI" target="_blank">GitHub</a> or from the PowerShell gallery, if you have WMF5 installed, by running **Install-Module WAPTenantPublicAPI**.

### A look inside the module

Almost all functions in the module rely heavily on the <a href="https://technet.microsoft.com/en-us/library/hh849971.aspx" target="_blank">Invoke-RestMethod</a> cmdlet. Microsoft is using REST all over the place these days which makes it really easy to start using PowerShell against Web APIs. All you need is a little pointer on what URLs to go after and what headers are expected and in no time you are actually interfacing and doing all kind of cool stuff.

<a href="https://technet.microsoft.com/en-us/library/hh849971.aspx" target="_blank">Invoke-RestMethod</a> and <a href="https://technet.microsoft.com/en-us/library/hh849901.aspx" target="_blank">Invoke-WebRequest</a> have been around since PowerShell 3.0 and as they form the basis of everything done in this module, I targeted for 3.0 compatibility. Unfortunately, I quickly stumbled upon a bug in the 3.0 versions of these cmdlets. As shown in the previous post, we use the following header format:

```
@{
    Authorization = "Bearer $Token"
    'x-ms-principal-id' = $Credential.UserName
    Accept = 'application/json'
}
```


The Accept part of the header tells the Azure Pack API to return response data as JSON. We want this as the API returns a lot more data when JSON is used. In PowerShell 3.0 this accept header causes an error to be thrown which has been logged on Connect as a bug: <a href="https://connect.microsoft.com/PowerShell/feedback/details/757249/invoke-restmethod-accept-header" target="_blank">Invoke-RestMethod Accept header</a>. Therefore, I decided to require PowerShell 4.0 as the bug was fixed in that release. I could have gone the long route around the issue and create the functionality myself but I figured as the intent is to have a DSC resource, 4.0 would be required anyway.

To force the PowerShell 4.0 requirement, I added a requires statement in the psm1 file (in case someone decides to load the module directly from this file)

```powershell
#requires -version 4
```


and defined the minimal PowerShellVersion in the module’s manifest (psd1) file.

```powershell
# Minimum version of the Windows PowerShell engine required by this module
PowerShellVersion = '4.0'
```

### Get-WAPToken

For Windows Azure Pack there can be 2 providers for the tokens. Either you have the inbox authentication site which generates the JWT tokens if successfully authenticated against the ASP.net membership provider or you have ADFS generating them from an external identity source like Active Directory or Azure Active Directory.  Luckily the Azure Pack team made this one really simple for me. When you install the Azure Pack PowerShell API (a component of Windows Azure Pack installation) a folder will be created “C:\Program Files\Management Service\MgmtSvc-PowerShellAPI\Samples\Authentication” which contain some PowerShell example scripts to get a token from different providers. I took these example scripts and combined them into the Get-WAPToken function.

The scripts relies on the <a href="https://msdn.microsoft.com/en-us/library/system.servicemodel(v=vs.110).aspx" target="_blank">System.ServiceModel</a> and <a href="https://msdn.microsoft.com/en-us/library/gg145031(v=vs.110).aspx" target="_blank">System.IdentityModel</a> assemblies to be loaded. As I created a function I did not want these assemblies to be loaded every time the function was called so I decided to load these as part of the module being imported.

```powershell
Add-Type -AssemblyName 'System.ServiceModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'
Add-Type -AssemblyName 'System.IdentityModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'
```


The function will set the Token and Header variables in the “parent” scope. This makes them available for subsequent functions to consume directly instead of the need to pass the data as parameter values.

```powershell
Set-Variable -Name Headers -Scope 1 -Value @{
Authorization = "Bearer $Token"
'x-ms-principal-id' = $Credential.UserName
Accept = 'application/json'
}
Set-Variable -Name Token -Value $token -Scope 1
```

I chose this method because almost all subsequent functions need this data. At first I took an alternative route and enriched the output object of functions with this data so I could pipe into the next function without re-specifying the values. In the end this proved to become really messy so I decided to divert from the approach.

On a side note: <a href="http://hindenes.com/trondsworking/" target="_blank">Trond Hindenes</a> suggested using variables instead of enriched objects through a GitHub <a href="https://github.com/bgelens/WAPTenantPublicAPI/issues/1" target="_blank">issue</a> on my project. How cool is that! Making the move to Git and GitHub is proving to be really valuable so I encourage you, if you are not already doing so, to start looking into source control.

### Connect-WAPAPI

This function is only used to set variables in the parent scope. We don’t actually connect to Azure Pack but try to see if we get results on a URL. If results are returned, the variables will be set. If an error is thrown instead, the variables will be nulled.

When the IgnoreSSL switch is set, the variable IgnoreSSL in the parent scope is set to true which means all subsequent functions will ignore certificate errors.

```powershell
if ($IgnoreSSL) {
    Write-Warning -Message 'IgnoreSSL switch defined. Certificate errors will be ignored!'
    #Change Certificate Policy to ignore
    IgnoreSSL  
    Set-Variable -Name IgnoreSSL -Value $IgnoreSSL -Scope 1
}

PreFlight

$TestURL = '{0}:{1}/subscriptions/' -f $URL,$Port
Write-Verbose -Message "Constructed Connection URL: $TestURL"
$Result = Invoke-WebRequest -Uri $TestURL -Headers $Headers -UseBasicParsing -ErrorVariable 'ErrCon'
if ($Result) {
    Write-Verbose -Message 'Successfully connected'
    Set-Variable -Name PublicTenantAPIUrl -Value $URL -Scope 1
    Set-Variable -Name Port -Value $Port -Scope 1
} else {
    Write-Verbose -Message 'Connection unsuccessfull' -Verbose
    Set-Variable -Name PublicTenantAPIUrl -Value $null -Scope 1
    Set-Variable -Name Port -Value $null -Scope 1
    throw $ErrCon
}
```

### PreFlight (helper function)

You might have seen PreFlight in the previous function code. PreFlight is a private function which means it is not exported to the user when the module is loaded. The module exports functions based on the presence of the WAP prefix (<verb><dash>**WAP**<Noun>), everything else is considered private.


```powershell
function PreFlight {
    [CmdletBinding()]
    param (
        [Switch] $IncludeConnection,
        [Switch] $IncludeSubscription
    )
    Write-Verbose -Message 'Validating Token Acquired'
    if (($null -eq $Token) -or ($null -eq $Headers)) {
        throw 'Token was not acquired, run Get-WAPToken first!'
    }

    Write-Verbose -Message 'Validating Token not expired'
    if (!(TestJWTClaimNotExpired -Token $Token)) {
        throw 'Token has expired, fetch a new one!'
    }

    if ($IncludeConnection) {
        Write-Verbose -Message 'Validating if connection is set'
        if ($null -eq $PublicTenantAPIUrl) {
            throw 'No connection has been made to API yet, run Connect-WAPAPI first!'
        }
    }

    if ($IncludeSubscription) {
        Write-Verbose -Message 'Validating if subscription is selected'
        if ($null -eq $Subscription) {
            throw 'No Subscription has been selected yet, run Select-WAPSubscription first!'
        }
    }
}
```
Because this helper function exists, it saves me from writing a lot of redundant code in other functions as almost all functions need to deal with these checks. These functions now just call this helper function and dependent on their functionality, provide additional switches to it so additional checks are done.</span>The functions job is to check if variables have been assigned data. It throws errors if these are still null. Also, if a token has been acquired, another helper function is called to check if the token is still valid.

Connect-WAPAPI provides no switches, so only the presence of the Token and Headers variables are checked and the Token is checked for expiration. When a function relies on Connect-WAPAPI to have already ran successfully, it will add the –IncludeConnection switch. And when a function relies on a subscription to be selected as current, it will add the –IncludeSubscription switch.

### TestJWTClaimNotExpired (Check token Expiration helper function)

This helper function is actually nice to see entirely so I can give some details of what we do here.


```powershell
function TestJWTClaimNotExpired {
    param (
        [Parameter(Mandatory,
                   ValueFromPipeline,
                   ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [String] $Token
    )
    #based on functions by Shriram MSFT found on technet: https://gallery.technet.microsoft.com/JWT-Token-Decode-637cf001
    process {
        try {
            if ($Token.split('.').count -ne 3) {
                throw 'Invalid token passed, run Get-WAPToken to fetch a new one'
            }

            $TokenData = $token.Split('.')[1] | ForEach-Object -Process {
                $data = $_ -as [String]
                $data = $data.Replace('-', '+').Replace('_', '/')
                switch ($data.Length % 4) {
                    0 { break }
                    2 { $data += '==' }
                    3 { $data += '=' }
                    default { throw New-Object -TypeName ArgumentException -ArgumentList ('data') }
                }
                [System.Text.Encoding]::UTF8.GetString([convert]::FromBase64String($data)) | ConvertFrom-Json
            }

            #JWT Reference Time
            $Ref = [datetime]::SpecifyKind((New-Object -TypeName datetime -ArgumentList ('1970',1,1,0,0,0)),'UTC')
            #UTC time right now - Reference time gives amount of seconds to check against
            $CheckSeconds = [System.Math]::Round(([datetime]::UtcNow - $Ref).totalseconds)
            if ($TokenData.exp -gt $CheckSeconds) {
                Write-Output -InputObject $true
            } else {
                Write-Output -InputObject $false
            }
        } catch {
            Write-Error -ErrorRecord $_
        }
    }
}
```
As you can see, I made a remark I borrowed some code from <a href="https://gallery.technet.microsoft.com/JWT-Token-Decode-637cf001" target="_blank">Shriram</a> who actually created a set of functions to convert a JWT token into a PSCustomObject so it can be analyzed. I modified it to the module’s needs and created a helper function from it. This helper function is called for every API interacting function (PreFlight) to check up front if the JWT token is not expired. This way it is prevented that the API itself throws any error related to the token’s lifetime and we can handle these errors in a uniform way.

A token looks like this:

![](/images/wapmod1.png)

Basically the JWT token exists out of 3 parts separated by a dot (.).

![](/images/wapmod2.png)

The second part ([1]) contains the claim and expiration information. So we are only interesting in this part for this helper function.

The replacing (or normalization if you will) and additions of ‘=’ or “==” happens because the JWT tokens generated by the Azure Pack Authentication site are not entirely the same as those generated by ADFS. Because of this, the Base64 to String conversion would trip if the text was not normalized.

Once the second part of the JWT token is converted it can be interpreted and looks like this:

![](/images/wapmod3.png)

As you can see my UPN claim is in there as well as a number defining my expiration time (exp).

JWT token validity is checked by calculating the amount of seconds which have passed from exactly the moment it became the first of January 1970 UTC time. So I set this as the reference time.

```powershell
$Ref = [datetime]::SpecifyKind((New-Object -TypeName datetime -ArgumentList ('1970',1,1,0,0,0)),'UTC')
```


Next we need to calculate the amount of seconds which have passed since then.

```powershell
$CheckSeconds = [System.Math]::Round(([datetime]::UtcNow - $Ref).totalseconds)
```


Now we know the amount of seconds passed, all that needs to be done is to check if the exp value of the claim is greater than the amount that passed since 1970. If this is the case, the token is still valid and we return a Boolean of true or else a Boolean of false.

```powershell
if ($TokenData.exp -gt $CheckSeconds) {
    Write-Output -InputObject $true
} else {
    Write-Output -InputObject $false
}
```


### Get-WAPSubscription and Select-WAPSubscription

When we look at the URL constructions of other functions you will see that the subscription id is in every one of them:

| **Function**           | **Url**                                                      |
| ---------------------- | ------------------------------------------------------------ |
| *Get-WAPGalleryVMRole* | ‘{0}:{1}/{2}/Gallery/GalleryItems/$/MicrosoftCompute.VMRoleGalleryItem?api-version=2013-03’ -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId |
| *Get-WAPVMRoleOSDisk*  | ‘{0}:{1}/{2}/services/systemcenter/vmm/VirtualHardDisks’ -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId |
| *Get-WAPVMNetwork*     | ‘{0}:{1}/{2}/services/systemcenter/vmm/VMNetworks’ -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId |
| *Get-WAPCloudService*  | ‘{0}:{1}/{2}/CloudServices?api-version=2013-03’ -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId |
| *Get-WAPVMRole*        | ‘{0}:{1}/{2}/CloudServices/{3}/Resources/MicrosoftCompute/VMRoles?api-version=2013-03’ -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId,$CloudServiceName |

This makes sense as the subscription governs everything what has been made available to the user by its association with a plan and potential add-ons.


```powershell
$URL = '{0}:{1}/subscriptions/' -f $PublicTenantAPIUrl,$Port
Write-Verbose -Message "Constructed Subscription URL: $URL"
$Subscriptions = Invoke-RestMethod -Uri $URL -Headers $Headers -Method Get
foreach ($S in $Subscriptions) {
    if ($PSCmdlet.ParameterSetName -eq 'Name' -and $S.SubscriptionName -ne $Name) {
        continue
    }
    if ($PSCmdlet.ParameterSetName -eq 'Id' -and $S.SubscriptionId -ne $Id) {
        continue
    }

    $S.Created = [datetime]$S.Created
    Add-Member -InputObject $S -MemberType AliasProperty -Name 'Subscription' -Value SubscriptionId
    $S.PSObject.TypeNames.Insert(0,'WAP.Subscription')
    Write-Output -InputObject $S
}
```
When you look at the code of Get-WAPSubscription you see the basis for almost all of the other functions. It first constructs the URL to target, then it runs Invoke-RestMethod with the Get method, capturing the results. By default, most functions have a DefaultParameterSetName of ‘List’. The List parameter set is actually not implemented but this makes it really clear to the coder wat is the intention when no parameters have been provided by the end user. As you can see, filtering on the output is done based on the parameter set in use. When the parameter set in use is the List parameter set, nothing is filtered and everything is returned to the pipeline. When the parameter set is Name on the other hand, only subscriptions which have the correct name will be passed on the pipeline, skipping everything else (the continue keyword moves to the next object in the foreach loop effectively stopping execution on the current object).

Once the desired subscription is found, we need to provide the other functions with its subscription Id by default. Like with the Headers, Token, API URL and other data, the subscription object is captured in a parent level variable. This is done by Select-WAPSubscription function.

```powershell
if ($input.count -gt 1) {
    throw 'Only 1 subscription can be selected. If passed from Get-WAPSubscription, make sure only 1 subscription object is passed on the pipeline'
}

if (!($Subscription.pstypenames.Contains('WAP.Subscription'))) {
    throw 'Object bound to Subscription parameter is of the wrong type'
}

Write-Verbose -Message "Setting current subscription to $($Subscription | Out-String)"
Set-Variable -Name Subscription -Value $Subscription -Scope 1
```

Select-WAPSubscription does support pipeline input but does not have a process block. This is done so we are certain the user is absolutely sure which subscription gets selected. The default available input variable is checked for its count and if it’s greater than 1, an error is thrown notifying the user to be more explicit.

When a usr wants to see which subscription is set, Get-WAPSubscription can be used with the –Current switch. This will output the content of the subscription variable.

### Get-WAPGalleryVMRole

```powershell
$URI = '{0}:{1}/{2}/Gallery/GalleryItems/$/MicrosoftCompute.VMRoleGalleryItem?api-version=2013-03' -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId
Write-Verbose -Message "Constructed Gallery Item URI: $URI"
$GalleryItems = Invoke-RestMethod -Uri $URI -Headers $Headers -Method Get

foreach ($G in $GalleryItems.value) {
    if ($PSCmdlet.ParameterSetName -eq 'Name' -and $G.Name -ne $Name) {
        continue
    }
    if ($Version -and $G.Version -ne $Version) {
        continue
    }

    $GIResDEFUri = '{0}:{1}/{2}/{3}/?api-version=2013-03' -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId,$G.ResourceDefinitionUrl
    Write-Verbose -Message "Acquiring ResDef from URI: $GIResDEFUri"
    $ResDef = Invoke-RestMethod -Uri $GIResDEFUri -Headers $Headers -Method Get

    $GIViewDefUri = '{0}:{1}/{2}/{3}/?api-version=2013-03' -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId,$G.ViewDefinitionUrl
    Write-Verbose -Message "Acquiring ViewDef from URI: $GIResDEFUri"
    $ViewDef = Invoke-RestMethod -Uri $GIViewDefUri -Headers $Headers -Method Get

    Add-Member -InputObject $G -MemberType NoteProperty -Name ResDef -Value $ResDef
    Add-Member -InputObject $G -MemberType NoteProperty -Name ViewDef -Value $ViewDef
    $G.PublishDate = [datetime]$G.PublishDate
    $G.PSObject.TypeNames.Insert(0,$G.'odata.type')
    Write-Output -InputObject $G
}
```
When looking at the code for Get-WAPGalleryVMRole you can see how we ended up with all the information we needed to construct our ResDef JSON with.

At first the URL is constructed to acquire the GalleryItems. Every item which is returned has a ResourceDefinitionUrl and ViewDefinitionUrl property. The API effectively told us where to find the info we need to deploy the VM Role with. Thankfully making use of the information, two other URLs are constructed and called. The information which is returned is added to the output object by making use of the Add-Member cmdlet before outputting the object on the pipeline. Now other functions, like Get-WAPVMRoleOSDisk and New-WAPVMRoleParameterObject, can make use of the all this data.

![](/images/wapmod4.png)

![](/images/wapmod5.png)

### New-WAPVMRoleParameterObject

Now we have the data which came with the Gallery Item object (and some other functions, see <a href="http://104.131.21.239/2015/08/07/journey-to-a-windows-azure-pack-vm-role-dsc-resource-powershell-module-usage/" target="_blank">part 2</a> of this series), we need to fill in the missing (user specific) data needed to deploy the VM Role. What needs to filled in is mandatorily exposed to the View Definition so I created this function, New-WAPVMRoleParameterObject, to deal with this task.

![](/images/wapmod6.png)

First the View Definition parameters are captured in a variable. Then each parameter is going through a foreach loop. If the -Interactive switch is enabled by the user, he will be prompted to provide values depending on the property to be of the “Option” or “Credential” type and for everything that will be left blank after not matching any of the elseif clauses. When it is not run in “interactive mode”, everything missing will have null as a value.

### New-WAPVMRoleDeployment

I want to close this post with the function that handles the most important task, the deployment.


```powershell
function New-WAPVMRoleDeployment {
    <# .SYNOPSIS Deploys VM Role to a Cloudservice using Azure Pack TenantPublic or Tenant API. .PARAMETER CloudServiceName The name of the cloud service to provision to. If it does not exist, it will be created. // Removed the rest to shorten code in blog post #>
    [CmdletBinding(SupportsShouldProcess=$true)]
    [OutputType([PSCustomObject])]
    param (
        [Parameter(Mandatory)]
        [ValidateNotNull()]
        [PSCustomObject] $VMRole,
    [Parameter(Mandatory)]
    [ValidateNotNull()]
    [PSCustomObject] $ParameterObject,

    [Parameter(Mandatory,
               ValueFromPipelineByPropertyName)]
    [Alias('Name','VMRoleName')]
    [ValidateNotNullOrEmpty()]
    [String] $CloudServiceName
)
process {
    $ErrorActionPreference = 'Stop'

    if (!($VMRole.pstypenames.Contains('MicrosoftCompute.VMRoleGalleryItem'))) {
        throw 'Object bound to VMRole parameter is of the wrong type'
    }

    if (!($ParameterObject.pstypenames.Contains('WAP.ParameterObject'))) {
        throw 'Object bound to ParameterObject parameter is of the wrong type'
    }

    $ParameterObject | Get-Member -MemberType Properties | ForEach-Object -Process {
        if ($null -eq $ParameterObject.($_.name)) {
            throw "ParameterObject property: $($_.name) is NULL"
        }
    }

    try {
        if ($IgnoreSSL) {
            Write-Warning -Message 'IgnoreSSL defined by Connect-WAPAPI, Certificate errors will be ignored!'
            #Change Certificate Policy to ignore
            IgnoreSSL
        }

        PreFlight -IncludeConnection -IncludeSubscription

        if ($PSCmdlet.ShouldProcess($CloudServiceName)) {
            Write-Verbose -Message "Testing if Cloudservice $CloudServiceName exists"

            if (!(Get-WAPCloudService -Name $CloudServiceName)) {
                Write-Verbose -Message "Creating Cloudservice $CloudServiceName as it does not yet exist"
                New-WAPCloudService -Name $CloudServiceName | Out-Null
                $New = $true
            } else {
                $New = $false
            }

            if (!$New) {
                Write-Verbose -Message "Testing if VMRole does not already exist within cloud service"
                if (Get-WAPCloudService -Name $CloudServiceName | Get-WAPVMRole) {
                    throw "There is already a VMRole deployed to the CloudService $CloudServiceName. Because this function mimics portal experience, only one VM Role is allowed to exist per CloudService"
                }
            }

            #Add ResDefConfig JSON to Dictionary
            $ResDefConfig = New-Object -TypeName 'System.Collections.Generic.Dictionary[String,Object]'
            $ResDefConfig.Add('Version',$VMRole.version)
            $ResDefConfig.Add('ParameterValues',($ParameterObject | ConvertTo-Json))

            # Set Gallery Item Payload Info
            $GIPayload = @{
                InstanceView = $null
                Substate = $null
                Name = $CloudServiceName
                Label = $CloudServiceName
                ProvisioningState = $null
                ResourceConfiguration = $ResDefConfig
                ResourceDefinition = $VMRole.ResDef
            }

            # Convert Gallery Item Payload Info To JSON
            $GIPayloadJSON = ConvertTo-Json -InputObject $GIPayload -Depth 10

            # Deploy VM Role to cloudservice
            $URI = '{0}:{1}/{2}/CloudServices/{3}/Resources/MicrosoftCompute/VMRoles/?api-version=2013-03' -f $PublicTenantAPIUrl,$Port,$Subscription.SubscriptionId,$CloudServiceName
            Write-Verbose -Message "Constructed VMRole Deploy URI: $URI"

            Write-Verbose -Message "Starting deployment of VMRole $VMRoleName to CloudService $CloudServiceName"
            $Deploy = Invoke-RestMethod -Uri $URI -Headers $Headers -Method Post -Body $GIPayloadJSON -ContentType 'application/json'
            $Deploy.PSObject.TypeNames.Insert(0,'WAP.VMRole')
            Write-Output -InputObject $Deploy
        }
    } catch {
        if ($New) {
             Get-WAPCloudService -Name $CloudServiceName | Remove-WAPCloudService -Force
        }
        Write-Error -ErrorRecord $_
    } finally {
        #Change Certificate Policy to the original
        if ($IgnoreSSL) {
            [System.Net.ServicePointManager]::CertificatePolicy = $OriginalCertificatePolicy
        }
    }
}
}
```
The input objects are checked if they contain the correct type information. I added type information to a lot of objects so I could do exactly this. A side benefit I also utilize is that I can use format.ps1xml because of this as well. Many objects returned by the API exist out of a lot of properties which are not directly valuable for the user so I defined custom list views in the format.ps1xml file.

![](/images/wapmod7.png)

This results in the following view:

![](/images/wapmod8.png)

Else it would have looked like this:

![](/images/wapmod9.png)

Because the module emulates portal behavior, the subscriptions is checked if a cloud service already exists with the name specified by the user. If this is the case, the cloud service is checked to contain a VM Role deployment and an error is thrown when this is the case. An empty cloud service is fine to continue. When no cloud service exists with the provided name, it is created on the spot.

Next the body of the API call is created.

```powershell
#Add ResDefConfig JSON to Dictionary
$ResDefConfig = New-Object -TypeName 'System.Collections.Generic.Dictionary[String,Object]'
$ResDefConfig.Add('Version',$VMRole.version)
$ResDefConfig.Add('ParameterValues',($ParameterObject | ConvertTo-Json))

# Set Gallery Item Payload Info
$GIPayload = @{
    InstanceView = $null
    Substate = $null
    Name = $CloudServiceName
    Label = $CloudServiceName
    ProvisioningState = $null
    ResourceConfiguration = $ResDefConfig
    ResourceDefinition = $VMRole.ResDef
}

# Convert Gallery Item Payload Info To JSON
$GIPayloadJSON = ConvertTo-Json -InputObject $GIPayload -Depth 10
```

Then a hash table is constructed which will form the actual body. It contains the justly created dictionary object, original resource definition and the cloud service name provided by the user. Finally, the hash table is converted into JSON (this would be the resdef needed by the Azure New-WAPackVMRole cmdlet).First a dictionary object is created and both the Gallery Item (VMRole) version and the generated parameter object is added to its properties respectively.

Finally, the VM Role deployment is started and a resulting object is put on the pipeline.

```powershell
$Deploy = Invoke-RestMethod -Uri $URI -Headers $Headers -Method Post -Body $GIPayloadJSON -ContentType 'application/json'
$Deploy.PSObject.TypeNames.Insert(0,'WAP.VMRole')
Write-Output -InputObject $Deploy
```


You might have noticed that these are not all the functions available in the module. I think the most important ones are described here that might give you an idea of what investment was needed to create all this.

Besides this, the reason for this series is to show you how easy it is to create a DSC resource on top of an already developed module and to hopefully make you aware where your investments should be.

I could have created a DSC resource containing all the code now embodied into this module but then this code would only be usable when a VM Role would be deployed through DSC. Now, because I developed the code as a normal PowerShell module first, the code can not only be used by the DSC resource, but by many other applications as well (e.g. PowerShell session, SMA, etc).

In the next (and final) post, I’ll show you how I’ve developed a DSC resource on top of the PowerShell module.

[1]: /2015/07/28/journey-to-a-windows-azure-pack-vm-role-dsc-resource/
[2]: /2015/08/07/journey-to-a-windows-azure-pack-vm-role-dsc-resource-powershell-module-usage/
[3]: /2015/09/22/part-4-journey-to-a-windows-azure-pack-vm-role-dsc-resource-the-dsc-resource/