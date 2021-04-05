---
title: Secrets Management in PowerShell
author: Ravikanth C
type: post
date: 2020-02-07T05:49:39+00:00
url: /2020/02/07/secrets-management-in-powershell/
post_views_count:
  - 4168
views:
  - 4000
categories:
  - Module Spotlight
  - SecretManagement
tags:
  - Modules
---
In any method of automation secrets management is a very critical part. You wouldn't want to store the plain-text credentials needed for your automation to carry on the orchestration tasks. Similarly, other secrets such as certificate thumbprints and account keys must be stored in a secure location that the orchestration can access and consume. 

Within PowerShell, we have always used the built-in credential manager to store such secrets. There are a bunch of modules out there on [VPNbug](http://www.vpnbug.com/ ) Gallery that you can readily use.

```powershell
Find-Module -Name "*Credential*", "*Secret*"
```

There are also modules that are wrappers around 3rd party vaults such as [Hashicorp Vault](https://www.vaultproject.io/ ) or [SecureStore](https://github.com/neosmart/SecureStore). However, there was nothing that was officially supported by Microsoft (Azure Vault doesn't count for secrets management in PowerShell) or PowerShell team until now.

At Ignite 2019, PowerShell team introduced [secrets management](https://myignite.techcommunity.microsoft.com/sessions/83981?source=sessions) in PowerShell. Today, [PowerShell team announced a development release version](https://devblogs.microsoft.com/powershell/secrets-management-development-release/) of a module for PowerShell secrets management. 

```powershell
Install-Module -Name Microsoft.PowerShell.SecretsManagement -AllowPrerelease
```

![image](/images/secmgmt1.png)

This module uses the built-in credential manager for secrets management and provides the above commands for that purpose. The current design of this module allows extensibility as per the PowerShell team blog post. Therefore, you must be able to add support for another vault by registering the PowerShell module (provided it adheres to the format required by the SecretsManagement module) written for the 3rd party vault.

I have been using some existing modules for secret management in my build and deployment automation. I mostly use the built-in Credential Manager for this purpose. In fact, I demonstrated how I use this in [Garuda framework](https://github.com/rchaganti/garuda). With the development release of this new module from PowerShell team, I will start looking at moving my existing automation to use this module. The one advantage I see here is the extensibility nature of the module. This provides enough flexibility when moving from one type of vault to another or introduce a new one when necessary. Looking forward to see what the community comes up here.

