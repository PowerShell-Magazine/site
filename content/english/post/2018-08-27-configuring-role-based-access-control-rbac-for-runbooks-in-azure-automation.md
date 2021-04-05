---
title: Configuring role-based access control (RBAC) for runbooks in Azure Automation
author: Jan Egil Ring
type: post
date: 2018-08-27T16:00:58+00:00
url: /2018/08/27/configuring-role-based-access-control-rbac-for-runbooks-in-azure-automation/
post_views_count:
  - 7888
views:
  - 8025
categories:
  - Azure
tags:
  - Azure Automation
  - Azure

---
When setting up runbooks in Azure Automation to invoke automation of a process through PowerShell or Python runbooks, the ability to assign permissions per runbook for user types such as power users is something we missed in the early days of Azure Automation.

When working with permissions in Azure, we have the concept of RBAC available:

_Role-based access control (RBAC) enables access management for Azure resources. Using RBAC, you can segregate duties within your team and grant only the amount of access to users, groups, and applications that they need to perform their jobs._

During the past year, the ability to [configure RBAC for runbooks][1] was added to the service.

When I initially tested this feature, I granted 1 single test user permissions to an existing user.

```powershell
Connect-AzureRmAccount
Connect-AzureAD

#Resource Group name for the Automation Account
$rgName = "AutomationWestEurope-Rg"

#Name of the Automation Account
$automationAccountName ="AutomationWestEurope"

#Name of the runbook
$rbName = "Invoke-RobocopyBackup"
$userId = (Get-AzureADUser -ObjectId 'demo.user@powershell.no').ObjectId 

#Gets the Automation Account resource
$aa = Get-AzureRmResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts" -ResourceName $automationAccountName

#Get the runbook resource
$rb = Get-AzureRmResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts/runbooks" -ResourceName "$automationAccountName/$rbName"

#The Automation Job Operator role only needs to be run once per user
New-AzureRmRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Job Operator" -Scope $aa.ResourceId

#Adds the user to the Automation Runbook Operator role to the runbook scope
New-AzureRmRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Runbook Operator" -Scope $rb.ResourceId
```

I then logged into the Azure subscription using that user. As expected, the only resource the user has access to is the single runbook where permissions are assigned:

![](/images/aa1.png)

This means the user is unable to see the whole Automation account, including resources such as global variables, credentials and other shared resources.

After starting a runbook job, everything looked like expected, except for the Output stream being unavailable (“No access”):

![image](/images/aa2.png)

After reaching out to the Automation team, I got in touch with Chris Sanders which determined that the _Microsoft.Automation/automationAccounts/jobs/output/read_ permission seemed to be missing.

I was recently asked to test again, since the bug is now fixed.

This time, the required permissions was in place and the Output stream was visible:

![](/images/aa3.png)

The ability to assign runbook permissions for users and groups is a very useful feature, making it possible to use the Azure Portal as the user interface for an automated process. It is then possible to perform tasks without granting the end user permissions directly to backend services such as a SQL database or local administrator permissions on a server.

[1]: https://docs.microsoft.com/en-us/azure/automation/automation-role-based-access-control#configure-rbac-for-runbooks