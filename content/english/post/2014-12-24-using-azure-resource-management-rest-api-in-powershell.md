---
title: Using Azure Resource Management REST API in PowerShell
author: Ravikanth C
type: post
date: 2014-12-24T17:00:16+00:00
url: /2014/12/24/using-azure-resource-management-rest-api-in-powershell/
categories:
  - Azure
tags:
  - Azure

---
The [Azure Resource Manager PowerShell module][1] has a subset of functionality that the [resource management REST API][2] offers.

![](/images/azureapi1.png)

Specifically, the ARM PowerShell module does not include cmdlets to get the resource provider information. Also, note that the [ARM REST API requests must be authenticated using Azure Active Directory (AD)][3]. This article shows you how to authenticate to Azure AD using PowerShell and access the REST APIs.

Before you can start using ARM REST API in PowerShell, you need to first create an AD application and give permissions to access the service management API. These steps are detailed in the MSDN article <http://msdn.microsoft.com/en-us/library/azure/dn790557.aspx>.

We also need the Microsoft.IdentityModel.Clients.ActiveDirectory .NET assembly for creating an access token. This is the [Azure Active Directory Authentication library][4]. This can be downloaded from [NuGet.org][5]. We can do this using the nuget.exe. The following code snippet shows how to use nuget.exe.

```
Invoke-WebRequest -Uri 'https://oneget.org/nuget-anycpu-2.8.3.6.exe' -OutFile "${env:Temp}\nuget.exe"
Start-Process -FilePath "${env:Temp}\nuget.exe" -ArgumentList 'install Microsoft.IdentityModel.Clients.ActiveDirectory' -WorkingDirectory $env:Temp

Add-Type -Path "${env:Temp}\Microsoft.IdentityModel.Clients.ActiveDirectory.2.13.112191810\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
```

Once we have the assembly loaded,  we can use the _AuthenticationContext_ and then acquire a token for the REST API access. Before we proceed, we need the tenant ID, client ID of the application you created earlier and your Azure subscription ID. The client ID can be obtained from the application dashboard.

![](/images/azureapi2.png)

The tenant ID can be retrieved by running the _Get-AzureAccount_ cmdlet.

![](/images/3-1024x139.png)

The following script shows how to build the necessary authorization header for the REST API access.

```
$tenantId = 'tenant-id'
$clientId = 'client-id'
$subscriptionId = 'subscription-id'

$authUrl = "https://login.windows.net/${tenantId}"

$AuthContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]$authUrl

$result = $AuthContext.AcquireToken("https://management.core.windows.net/",
$clientId,
[Uri]"https://localhost",

[Microsoft.IdentityModel.Clients.ActiveDirectory.PromptBehavior]::Auto)

$authHeader = @{
'Content-Type'='application\json'
'Authorization'=$result.CreateAuthorizationHeader()
}
```

In the above snippet, the [_AcquireToken_][6] method gives us the access tokens. The _[Uri]&#8221;https://localhost&#8221;_ needs to be replaced with whatever you mentioned during the creation of application. When the [_AcquireToken_][6] method is executed, you may be prompted for the sign-in details as required.

From the _AcquireToken_ method output, we generate the required authorization header for accessing the REST API.

### List all Azure Resource Providers

The REST API for listing all resource providers is https://management.azure.com/subscriptions/{subscription-id}/providers?$skiptoken={skiptoken}&api-version={api-version}.

We can use the _Invoke-RestMethod_ cmdlet to access this REST endpoint. We need to use the _$authHeader_ created above with this cmdlet.

```
$allProviders = (Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/${subscriptionId}/providers?api-version=2014-04-01-preview" -Headers $authHeader -Method Get -Verbose).Value
```

![](/images/4-1024x439.png)

### Get Resource Provider Details

The REST API for getting details about a resource provider is https://management.azure.com/subscriptions/{subscription-id}/providers/{resource-provider-namespace}?api-version={api-version}.

<pre class="brush: powershell; title: ; notranslate" title="">$computeProvider = (Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/${subscriptionId}/providers/Microsoft.classicCompute?api-version=2014-04-01-preview" -Headers $authHeader -Method Get -Verbose)
</pre>
![](/images/5-1024x290.png)

What we have seen so far is only GET requests using the _Invoke-RestMethod_ cmdlet. Some REST API endpoints require POST method. One example we will see now is to register the subscription with a specific resource provider.

In the output showing a list of all providers, you see that my subscription is not registered with the Microsoft.Search resource provider. Let us see how we complete this registration using REST API in PowerShell.

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/${subscriptionId}/providers/Microsoft.Search/register?api-version=2014-04-01-preview" -Method Post -Headers $authHeader -Verbose
</pre>
![](/images/6-1024x115.png)

Once the registration is complete, you can see that in the all provider output.

![](/images/7-1024x413.png)

This is it. I hope you find this helpful! More on the Azure Resource Manager in future posts. Stay tuned.

[1]: http://azure.microsoft.com/en-us/documentation/articles/powershell-azure-resource-manager/
[2]: http://msdn.microsoft.com/en-us/library/azure/dn790568.aspx
[3]: http://msdn.microsoft.com/en-us/library/azure/dn790557.aspx
[4]: http://msdn.microsoft.com/en-us/library/azure/jj573266.aspx
[5]: https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/3.0.110281957-alpha
[6]: http://msdn.microsoft.com/en-us/library/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretoken.aspx