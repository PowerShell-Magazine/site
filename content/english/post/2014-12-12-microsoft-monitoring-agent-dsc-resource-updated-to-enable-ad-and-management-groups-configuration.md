---
title: Microsoft Monitoring Agent DSC resource updated to enable AD and management groups configuration
author: Ravikanth C
type: post
date: 2014-12-12T17:00:14+00:00
url: /2014/12/12/microsoft-monitoring-agent-dsc-resource-updated-to-enable-ad-and-management-groups-configuration/
categories:
  - PowerShell DSC
  - Azure
tags:
  - Azure
  - PowerShell DSC

---
A couple of weeks ago, I had announced a [custom DSC resource module to install and configure Microsoft Monitoring Agent][1] that is required to configure [Azure Operational Insights][2]. It was released with only Operational Insights enablement. Today I pushed a new version of this module to include management group configuration for connecting to on-premises Operations Manager deployment and Active Directory integration. With these two new resources, the cMMAgent DSC resource module is complete and contains six resources.

Microsoft Monitoring Agent ([cMMAgent][3]) DSC resource module contains the following resources.

<ul class="task-list">
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentInstall">cMMAgentInstall</a> is used to install Microsoft Monitoring Agent.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentProxyName">cMMAgentProxyName</a> is used to add or remove the proxy URL for the Microsoft Monitoring Agent configuration.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentProxyCredential">cMMAgentProxyCredential</a> is used to add, modify, or remove the credentials that need to be used to authenticate to a proxy configured using <em>cMMAgentProxyName</em> resource.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentOpInsights">cMMAgentOpInsights</a> is used to enable or disable Azure Operational Insights within the Microsoft Monitoring Agent. This can also be used to update the WorkspaceID and WorkspaceKey for connecting to Azure Operational Insights.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentAD">cMMAgentAD</a> is used to enable or disable Active Directory integration for the Microsoft Management Agent. By enabling AD integration, you can assign agent-managed computers to management groups.
  </li>
  <li>
    <a href="https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentManagementGroups">cMMAgentManagementGroups</a> DSC resource can be used to add or remove management groups. You can use this resource to update the action account credentials for the management agent service.
  </li>
</ul>

Along with the two new DSC resources there are a couple of breaking changes. I renamed _UpdateWorkspace_ property in _cMMAgentOpInsights_ resource and _UpdateCredential_ property of _cMMAgentProxyCredential_ resource to _Force_. This decision was taken to ensure that the resource design is inline with the recommended DSC best practices and patterns.

You can take a look at the [readme][4] on [Github][3] for a complete overview of each resource in the cMMAgent DSC module. Here is an explanation of the two new resources I added today.

#### Using cMMAgentAD resource

The [cMMAgentAD][5] resource can be used to enable or disable Active Directory (AD) integration. With AD Directory Services integration, the agent-managed computers can be automatically assigned to the Operations Manager management groups.

![](/images/cMMAgentAD.png)

There is only one property that is _EnableAD_. This is a Boolean property. Setting this to _True_ enables AD integration and disables otherwise.

<pre class="brush: powershell; title: ; notranslate" title="">cMMAgentAD MMAgentAD {
    EnableAD = $true
}
</pre>

#### [][6]{#user-content-using-cmmagentmanagementgroups-resource.anchor}Using cMMAgentManagementGroups resource

The cMMAgentManagementGroups resource enables you to add or remove Operations Manager management groups from the Microsoft Monitoring Agent configuration. Additionally, you can configure action account credentials for the agent service.

![](/images/cMMAgentManagementGroups.png)

The _ManagementGroupName_ and _ManagementServerName_ properties are mandatory. The _ManagementServerPort_ is optional and set to 5723, by default. This property need not be changed unless your Operations Manager implementation is customized.

<pre class="brush: powershell; title: ; notranslate" title="">cMMAgentManagementGroups MMAgentManagementGrup {
    ManagementGroupName = 'SCMgmtGroup'
    ManagementServerName = 'SCOM-1'
    Ensure = 'Present'
}
</pre>

You can specify the action account credentials for the management agent service. This needs to be a _PSCredential_ object. So, as per the DSC best practices, you must encrypt these credentials using certificates.

<pre class="brush: powershell; title: ; notranslate" title="">cMMAgentManagementGroups MMAgentManagementGrup {
    ManagementGroupName = 'SCMgmtGroup'
    ManagementServerName = 'SCOM-1'
    ActionAccountCredential = $Credential
    Ensure = 'Present'
}
</pre>

If you do not have certificate implementation in your test or development infrastructure, you can use DSC configuration data to allow plain text credentials.

```
$ActionAccountCredential = Get-Credential

$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'WMF5-1' }
   )
}

Configuration MMAgentConfiguration {
   Import-DscResource -ModuleName cMMAgent
  Node $AllNodes.NodeName {
     cMMAgentManagementGroups MMAgentManagementGrup {
        ManagementGroupName = 'SCMgmtGroup'
        ManagementServerName = 'SCOM-1'
        ActionAccountCredential = $ActionAccountCredential
        Ensure = 'Present'
     }
   }
}
```

Once you add a management group, if you need to update the action account credentials, you can use the _Force_ property. When you set _Force_ property to True, the action account credentials get updated. The value of _Ensure_ property has no meaning when using the _Force_ property. Also, when using _Force_, the _actionAccountCredential_ property must be set.

```
$updatedActionAccountCredential = Get-Credential

$ConfigData = @{
   AllNodes = @(
      @{ NodeName = '*'; PsDscAllowPlainTextPassword = $true },
      @{ NodeName = 'WMF5-1' }
   )
}

Configuration MMAgentConfiguration {
   Import-DscResource -ModuleName cMMAgent
   Node $AllNodes.NodeName {
      cMMAgentManagementGroups MMAgentManagementGrup {
         managementGroupName = 'SCMgmtGroup'
         managementServerName = 'SCOM-1'
         actionAccountCredential = $updatedActionAccountCredential
         Force = $true
      }
   }
}
```

This is it. So, what is next? I will publish the DSC resource modules that I am developing to PowerShell Gallery so that you can use PowerShellGet module to download them to target systems. Stay tuned.

[1]: /2014/11/26/dsc-resource-module-for-microsoft-monitoring-agent-install-and-configuration-for-azure-operational-insights/
[2]: https://preview.opinsights.azure.com/
[3]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent
[4]: https://github.com/rchaganti/DSCResources/blob/master/cMMAgent/README.md
[5]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent/DSCResources/cMMAgentAD
[6]: https://github.com/rchaganti/DSCResources/tree/master/cMMAgent#using-cmmagentmanagementgroups-resource