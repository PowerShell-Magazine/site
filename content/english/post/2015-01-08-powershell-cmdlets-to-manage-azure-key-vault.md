---
title: PowerShell cmdlets to manage Azure Key Vault
author: Ravikanth C
type: post
date: 2015-01-09T04:12:34+00:00
url: /2015/01/08/powershell-cmdlets-to-manage-azure-key-vault/
views:
  - 10729
post_views_count:
  - 1919
categories:
  - Azure
  - News
tags:
  - Azure
  - News

---
Microsoft announced a public preview of [Azure Key Vault][1]. Azure Key Vault is a cloud-hosted HSM-backed service for managing cryptographic keys and other secrets used in your cloud applications.

To support the management of the keys and secrets, [Azure PowerShell tools is updated to version 0.8.13][2].

The following new cmdlets will be available in _AzureResourceManager_ mode.

<ul class="task-list">
  <li>
    Manage Keys: <ul class="task-list">
      <li>
        Add-AzureKeyVaultKey
      </li>
      <li>
        Get-AzureKeyVaultKey
      </li>
      <li>
        Set-AzureKeyVaultKey
      </li>
      <li>
        Backup-AzureKeyVaultKey
      </li>
      <li>
        Restore-AzureKeyVaultKey
      </li>
      <li>
        Remove-AzureKeyVaultKey
      </li>
    </ul>
  </li>

 Update: These cmdlets do not have the Key Vault creation functionality. This feature is provided as a sample script in a module called [KeyVaultManager][3]. You need to download this and create a vault before you can use any of the cmdlets in Azure PowerShell module.

More on how you can use these cmdlets in a later post!

[1]: http://blogs.technet.com/b/kv/archive/2015/01/08/azure-key-vault-making-the-cloud-safer.aspx
[2]: https://github.com/Azure/azure-powershell/releases/tag/v0.8.13-January2015
[3]: https://gallery.technet.microsoft.com/scriptcenter/Azure-Key-Vault-Powershell-1349b091