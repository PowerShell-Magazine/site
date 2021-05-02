---
title: DSC resource module for Microsoft Monitoring Agent install and configuration of Azure Operational Insights
author: Ravikanth C
type: post
date: 2014-11-26T17:00:49+00:00
url: /2014/11/26/dsc-resource-module-for-microsoft-monitoring-agent-install-and-configuration-for-azure-operational-insights/
categories:
  - Azure
  - PowerShell DSC
tags:
  - Azure
  - PowerShell DSC

---
Microsoft Monitoring Agent (MMA) is used to connect target systems to System Center Operations Manager or directly to Azure Operational Insights. This DSC resource module intends to build custom resources for installing and configuring MMA on target systems.

At present, there are four resources available within this module.

<ul class="task-list">
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentInstall">cMMAgentInstall</a> is used to install Microsoft Monitoring Agent.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentProxyName">cMMAgentProxyName</a> is used to add or remove the proxy URL for the Microsoft Monitoring Agent configuration.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentProxyCredential">cMMAgentProxyCredential</a> is used to add, modify, or remove the credentials that need to be used to authenticate to a proxy configured using cMMAgentProxyName resource.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentOpInsights">cMMAgentOpInsights</a> is used to enable or disable Azure Operational Insights within the Microsoft Monitoring Agent. This can also be used to update the WorkspaceID and WorkspaceKey for connecting to Azure Operational Insights.
  </li>
</ul>

This module is still work in-progress. I will add AD integration and management group configuration very soon.

I could have combined the resources into just a couple of them but that increases the complexity of the resource module. Therefore, I decided to go much granular and divide these into multiple resources. For example, the[cMMAgentProxyCredential][1] resource lets you not just add or remove credentials but also update the credentials, if required.

#### [][2]{#user-content-using-cmmagentinstall-resource.anchor}Using cMMAgentInstall resource

The cMMAgentInstall is a composite DSC resource. This can be used to install MMAgent in an unattended manner. Behind the scenes, this resource uses the Package DSC resource.

![](/images/MMAgentInstall.png)

The _Path_ property is used to specify a local folder path where MMAgent installer is stored. This can also be the HTTP location for downloading an installer. For example, the following function can be used to get the redirected URL from fwlink on Microsoft site.

```
Function Get-TrueURL {
   Param (
      [parameter(Mandatory)]
      $Url
   )
   $req = [System.Net.WebRequest]::Create($url)
   $req.AllowAutoRedirect=$false
   $req.Method="GET"

   $resp=$req.GetResponse()
   if ($resp.StatusCode -eq "Found") {
      return $resp.GetResponseHeader("Location")
   }
   else {
      return $resp.responseURI
   }
}
```

The fwlink to download MMAgent installer is <http://go.microsoft.com/fwlink/?LinkID=517476>.

<pre class="brush: powershell; title: ; notranslate" title="">$TrueUrl = Get-TrueUrl -Url 'http://go.microsoft.com/fwlink/?LinkID=517476'
</pre>

Now, this redirected URL can be given as value can be given as the value of _Path_ property. The Package resource downloads the installer before it attempts to install it on the target system.

The _WorkspaceID_ and the _WorkspaceKey_ can be retrieved from the [Azure Operational Insights preview portal][3] by navigating to Servers & Usage -> Configure.

![](/images/workspace.png)

Here is a configuration script that uses the cMMAgentInstall composite resource.

```
Configuration MMASetup {
   Import-DscResource -Module cMMAgent -Name cMMAgentInstall
   cMMAgentInstall MMASetup {
      Path = 'C:\OpInsights\MMASetup-AMD64.exe'
      Ensure = 'Present'
      WorkspaceID = 'your-Workspace-Id'
      WorkspaceKey = 'Your-Workspace-Key'
   }
}

MMASetup
```

#### Using cMMAgentProxyName resource

The [cMMAgentProxyName][5] resource can be used to add or remove the proxy URL to the Microsoft Monitoring Agent configuration.

![](/images/MMAgentproxy.png)

The only property here is the _ProxyName_ property. For this, you need to specify a proxy URL (either HTTP or HTTPS) with any port number as required. The following configuration script demonstrates this usage.

```
cMMAgentProxyName MMAgentProxy {
    ProxyName = 'https://moxy.in.com:3128'
    Ensure = 'Present'
}
```


#### Using cMMAgentProxyCredential resource

The [cMMAgentProxyName][5] resource only lets you configure the proxy URL that needs to be used to connect to the Azure Operational Insights service. However, if the proxy requires authentication, you can use the [cMMAgentProxyCredential][1]resource configure the same.

![](/images/cMMAgentProxycred.png)

When I was writing this resource module, there was a challenge in having only a _PSCredential_ type property as the Key property. The schema MOF was failing validation and therefore forced me to separate it into _ProxyUserName_ and_ProxyUserPassword_ properties. The _ProxyUserPassword_ is of type PSCredential and you can use the _Get-Credential_cmdlet to supply that. The user name you supply as a part of the PSCredential will not be used. Instead, the value supplied as an argument for the _ProxyUserName_ property will be used.

Note that DSC best practices recommend that you encrypt the credentials used in a configuration script. So, ideally, you should use certificates to encrypt and decrypt the credentials. For test and development purposes, however, you can use the plain text passwords and this can be achieved by using DSC configuration data. The following example demonstrates this.

```
$ProxyUserPassword = Get-Credential
$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'WMF5-1' }
   )
}

Configuration MMAgentConfiguration {
   Import-DscResource -ModuleName cMMAgent
   Node $AllNodes.NodeName {
      cMMAgentProxyCredential MMAgentProxyCred {
         ProxyUserName = 'ravikanth'
         ProxyUserPassword = $ProxyUserPassword
         Ensure = 'Present'
      }
   }
}
```

Once you set the proxy credentials, if you need to change the password alone within the credentials, you can use the_UpdateCredential_ property to force that change. When using _UpdateCredential_ property, setting Ensure=Absent is not valid.

```
$NewProxyUserPassword = Get-Credential

$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'WMF5-1' }
   )
}

Configuration MMAgentConfiguration {
   Import-DscResource -ModuleName cMMAgent
   Node $AllNodes.NodeName {
      cMMAgentProxyCredential MMAgentProxyCred {
         ProxyUserName = 'ravikanth'
         ProxyUserPassword = $NewProxyUserPassword
         UpdateCredential = $True
         Ensure = 'Present'
      }
   }
}
```

#### Using cMMAgentOpInsights resource

The [cMMAgentInstall][8] resource, by default, enables the Microsoft Monitoring Agent for Azure Operational Insights. This resource also configures the WorkspaceID and the WorkspaceKey required to connect to the Operational Insights service.

The [cMMAgentOpInsights][9] resource can be used to update the _WorkspaceID_ and _WorkspaceKey_, if required and also to disable Azure Operational Insights within the Microsoft Monitoring Agent.

![](/images/mmagentopinsights.png)

By setting both the _WorkspaceID_ and the _WorkspaceKey_ properties and _Ensure_ to _Present_, you can enable Azure Operational Insights and update the WorkspaceID and WorkspaceKey.

<pre class="brush: powershell; title: ; notranslate" title="">cMMAgentOpInsights MMAgentOpInsights {
    WorkspaceID = 'your-Workspace-ID'
    WorkspaceKey = 'your-Workspace-Key'
    Ensure = 'Absent'
}
</pre>

By setting _Ensure_ to _Absent_ along with required properties, Azure Operational Insights can be disable for the Microsoft Monitoring Agent.

<pre class="brush: powershell; title: ; notranslate" title="">cMMAgentOpInsights MMAgentOpInsights {
   WorkspaceID = 'your-Workspace-ID'
   WorkspaceKey = 'your-Workspace-Key'
   Ensure = 'Absent'
}
</pre>

If you need to update only the WorkspaceKey, you can do that using the _UpdateWorkspace_ property.

<pre class="brush: powershell; title: ; notranslate" title="">cMMAgentOpInsights MMAgentOpInsights {
   WorkspaceID = 'your-Workspace-ID'
   WorkspaceKey = 'your-new-Workspace-Key'
   UpdateWorkspace = $true
}
</pre>

This is it for now. Stay tuned for more updates on this.

[1]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentProxyCredential
[2]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent#using-cmmagentinstall-resource
[3]: https://preview.opinsights.azure.com/
[4]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent#using-cmmagentproxyname-resource
[5]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentProxyName
[6]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent#using-cmmagentproxycredential-resource
[7]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent#using-cmmagentopinsights-resource
[8]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentInstall
[9]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentOpInsights