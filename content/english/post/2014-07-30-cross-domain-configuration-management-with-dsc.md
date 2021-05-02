---
title: Cross-domain configuration management with DSC
author: Ravikanth C
type: post
date: 2014-07-30T16:00:51+00:00
url: /2014/07/30/cross-domain-configuration-management-with-dsc/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC
---
Most of the DSC examples we have seen so far here or elsewhere focused on single domain scenarios. However, in a real-world deployment, we have mixed domains and even a few systems that are probably in a workgroup configuration. In this article, we will see what is needed to use DSC in a mixed domain environment.

We understand that DSC depends on CIM and WSMAN. When we push configuration to a target system, by default, we are using WinRM HTTP port (5895). We can configure HTTPS listeners too but let us keep that out for now. It is important to understand the configuration required to push configurations from a system in one domain to target systems in another domain. This scenario is what we refer as cross-domain configuration management.

For demonstration purposes, I will use the following configuration. I have two domains named _DSCDemo_ and _DSCDemo2_. These are two different AD forests. In this scenario, _DSCDemo_ is the account domain and _DSCDemo2_ is the resource domain.

![](/images/ad-forest1.png)

If we use default settings, a cross-domain configuration management using the _Start-DscConfiguration_ cmdlet fails with a WinRM error. We will use the following configuration script for all the examples in this article.

```
Configuration Demo {
   Node WSR2-4.DSCDemo2.lab {
       File TestFile {
          DestinationPath = "C:\Scripts\Test.txt"
          Contents = ""
          Ensure = "Present"
       }
   }
}
```


If we push this configuration from a system in _DSCDemo_ domain, it will fail.

![](/images/26-1024x222.png)

There are a couple of methods we can use to perform cross-domain configuration management:

  1. Using _-Credential_ parameter of the _Start-DscConfiguration_ cmdlet
  2. Creating a AD forest trust and configuring appropriate user permissions

### Using Credentials

Using the _-Credential_ parameter is the most easiest way to resolve the errors we see in mixed domain environments.

```
Start-DscConfiguration -Path .\Demo -Wait -Verbose -Credential (Get-Credential -UserName DSCDemo2\Administrator -Message "Password")
```

![](/images/33-1024x330.png)

However, the ideal method is to create a trust between the AD forests and configure appropriate permissions so that the domain administrators from the account domain can push configuration to systems in a resource domain.

### Using AD trust relationship

For this demo purpose, I created a two-way forest trust between the local and remote forests.

```
$Cred = Get-Credential
$RemoteForest = New-Object System.DirectoryServices.ActiveDirectory.DirectoryContext('Forest','DSCDemo2.lab',$cred.UserName,$Cred.GetNetworkCredential().Password)
$RemoteForestObject = [System.DirectoryServices.ActiveDirectory.Forest]::GetForest($RemoteForest)
$LocalForest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
$LocalForest.CreateTrustRelationship($RemoteForestObject,'Bidirectional')
```

![](/images/crossad.png)

Creating a forest trust alone isn&#8217;t enough. You need to ensure that the [name suffix routing is enabled][1] and the DNS server in the account domain is [configured to forward the name resolution requests][2] for all systems in the resource domain, if there is no shared root DNS.

When the forest trust is established and verified, we can configure the domain administrator user account permissions. This is what you need to perform:

  1. Create a Domain local security group in the resource domain (DSCDemo2 in our example).
  2. Add Domain Admins group from account domain as a member of domain local group created in step 1.
  3. Create a group policy in the resource domain to add the group created in step 1 as a member of local administrators group of all systems in the resource domain.

The first two steps are trivial and don&#8217;t need a lot of explanation. For the third step, I will show the configuration settings from my example.

![](/images/52-1024x336.png)

This is it. Once this GPO is enforced, we should be able to push configurations from the account domain to the resource domain without explicitly providing any credentials.

![](/images/61-1024x295.png)

Following the same steps, you can expand this to any number of domains in an enterprise environment. All this configuration is required only for pushing configuration. Pull configuration delivery model doesn't need these additional configuration steps. In a large enterprise Active Directory environment, pull configuration delivery is recommended way to implement DSC.

[1]: http://technet.microsoft.com/en-us/library/cc758181(v=ws.10).aspx
[2]: http://technet.microsoft.com/en-us/library/cc816856.aspx