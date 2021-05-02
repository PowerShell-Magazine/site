---
title: '#PSTip Show friendly GPO names with Get-ADOrganizationalUnit'
author: Jaap Brasser
type: post
date: 2014-07-25T18:00:25+00:00
url: /2014/07/25/pstip-show-friendly-gpo-names-with-get-adorganizationalunit/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When using the _Get-ADOrganizationalUnit_ cmdlet there is a property available, _LinkedGroupPolicyObjects_. Unfortunately, that property only displays the GUID of the Group Policy Objects. To make it easier to identify which GPOs are linked to an Organizational Unit the following function will gather the display names for these GPOs and place the names of the GPOs in a new property named ‘_FriendlyGPODisplayName_’:

```
Funtion Get-OUWithGPOLink {
   Get-ADOrganizationalUnit -Filter "Name –like '*'" -Properties name, distinguishedName, gpLink |
   Select-Object -Property *, @{
       Name = 'FriendlyGPODisplayName'
       Expression = {
           $_.LinkedGroupPolicyObjects | ForEach-Object {
             -join ([adsi]"LDAP://$_").displayName
           }
       }
   }
}
```


To illustrate what this means in practice I have included the difference between the output of _Get-ADOrganizationalUnit_ and _Get-OUWithGPOLink_:

![](/images/151.png)

Please note that this function still requires the PowerShell Active Directory module to be present on the system. The function utilizes ADSI to connect to the ADObject of the GPO to get the _DisplayName_ value. The display names are made available by utilizing _Select-Object_ with a calculated property.

The full version of this function is available and will be maintained in the TechNet Script Gallery, available [here][1].

[1]: http://gallery.technet.microsoft.com/Get-OUWithGPOLinks-List-02bfe340