---
title: '#PSTip List all Active Directory constructed attributes'
author: Shay Levy
type: post
date: 2013-04-25T18:00:00+00:00
url: /2013/04/25/pstip-list-all-active-directory-constructed-attributes/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
Active Directory has a special kind of attributes called Constructed attributes. Constructed attributes are not &#8220;real&#8221; attributes, they are not stored in the directory. Instead, their values are calculated (by a domain controller) from normal attributes (for read) and/or have effects on the values of normal attributes (for write). For example, a user object has constructed attributes such as _canonicalName_ and _distinguishedName_.

The following LDAP filter queries the Active Directory schema by using a bitwise filter to return only objects that match a particular bit being set. _1.2.840.113556.1.4.803_ is the [_LDAP\_MATCHING\_RULE\_BIT\_AND_][1] rule. The matching rule is true only if all bits from the property match the value. This rule is like the bitwise _AND_ operator (_1.2.840.113556.1.4.804_ is the [_LDAP\_MATCHING\_RULE\_BIT\_OR_][2] rule).

Using the bitwise _AND_ rule, we can determine if an attribute has the _FLAG\_ATTR\_IS_CONSTRUCTED_ bit set. The value of _FLAG\_ATTR\_IS_CONSTRUCTED_ is 4. We check that the _objectClass_ is an _attributeSchema_ and the attribute flags match the _FLAG\_ATTR\_IS_CONSTRUCTED_ value.

```
$FLAG_ATTR_IS_CONSTRUCTED=4

$filter = "(&(systemFlags:1.2.840.113556.1.4.803:=$FLAG_ATTR_IS_CONSTRUCTED)(ObjectClass=attributeSchema))"
Get-ADObject -SearchBase (Get-ADRootDSE).SchemaNamingContext -LDAPFilter $filter |
Select-Object Name,DistinguishedName | Sort-Object Name

Name                                        DistinguishedName
----                                        -----------------
Allowed-Attributes                          CN=Allowed-Attributes,CN=Schema,CN=Configuration,DC=doma...
Allowed-Attributes-Effective                CN=Allowed-Attributes-Effective,CN=Schema,CN=Configurati...
Allowed-Child-Classes                       CN=Allowed-Child-Classes,CN=Schema,CN=Configuration,DC=d...
Allowed-Child-Classes-Effective             CN=Allowed-Child-Classes-Effective,CN=Schema,CN=Configur...
ANR                                         CN=ANR,CN=Schema,CN=Configuration,DC=domain,DC=com
Attribute-Types                             CN=Attribute-Types,CN=Schema,CN=Configuration,DC=domain,...
Canonical-Name                              CN=Canonical-Name,CN=Schema,CN=Configuration,DC=domain,D...
Create-Time-Stamp                           CN=Create-Time-Stamp,CN=Schema,CN=Configuration,DC=domai...
DIT-Content-Rules                           CN=DIT-Content-Rules,CN=Schema,CN=Configuration,DC=domai...
Entry-TTL                                   CN=Entry-TTL,CN=Schema,CN=Configuration,DC=domain,DC=com
Extended-Attribute-Info                     CN=Extended-Attribute-Info,CN=Schema,CN=Configuration,DC...
Extended-Class-Info                         CN=Extended-Class-Info,CN=Schema,CN=Configuration,DC=dom...
From-Entry                                  CN=From-Entry,CN=Schema,CN=Configuration,DC=domain,DC=com
Modify-Time-Stamp                           CN=Modify-Time-Stamp,CN=Schema,CN=Configuration,DC=domai...
ms-DS-Approx-Immed-Subordinates             CN=ms-DS-Approx-Immed-Subordinates,CN=Schema,CN=Configur...
ms-DS-Auxiliary-Classes                     CN=ms-DS-Auxiliary-Classes,CN=Schema,CN=Configuration,DC...
ms-DS-isGC                                  CN=ms-DS-isGC,CN=Schema,CN=Configuration,DC=domain,DC=com
ms-DS-isRODC                                CN=ms-DS-isRODC,CN=Schema,CN=Configuration,DC=domain,DC=com
ms-DS-Is-User-Cachable-At-Rodc              CN=ms-DS-Is-User-Cachable-At-Rodc,CN=Schema,CN=Configura...
ms-DS-KeyVersionNumber                      CN=ms-DS-KeyVersionNumber,CN=Schema,CN=Configuration,DC=...
ms-DS-Local-Effective-Deletion-Time         CN=ms-DS-Local-Effective-Deletion-Time,CN=Schema,CN=Conf...
ms-DS-Local-Effective-Recycle-Time          CN=ms-DS-Local-Effective-Recycle-Time,CN=Schema,CN=Confi...
ms-DS-NC-Repl-Cursors                       CN=ms-DS-NC-Repl-Cursors,CN=Schema,CN=Configuration,DC=d...
ms-DS-NC-Repl-Inbound-Neighbors             CN=ms-DS-NC-Repl-Inbound-Neighbors,CN=Schema,CN=Configur...
ms-DS-NC-Repl-Outbound-Neighbors            CN=ms-DS-NC-Repl-Outbound-Neighbors,CN=Schema,CN=Configu...
ms-DS-Principal-Name                        CN=ms-DS-Principal-Name,CN=Schema,CN=Configuration,DC=do...
ms-DS-Quota-Effective                       CN=ms-DS-Quota-Effective,CN=Schema,CN=Configuration,DC=d...
ms-DS-Quota-Used                            CN=ms-DS-Quota-Used,CN=Schema,CN=Configuration,DC=domain...
ms-DS-Repl-Attribute-Meta-Data              CN=ms-DS-Repl-Attribute-Meta-Data,CN=Schema,CN=Configura...
ms-DS-Repl-Value-Meta-Data                  CN=ms-DS-Repl-Value-Meta-Data,CN=Schema,CN=Configuration...
ms-DS-Resultant-PSO                         CN=ms-DS-Resultant-PSO,CN=Schema,CN=Configuration,DC=dom...
ms-DS-Revealed-List                         CN=ms-DS-Revealed-List,CN=Schema,CN=Configuration,DC=dom...
ms-DS-Revealed-List-BL                      CN=ms-DS-Revealed-List-BL,CN=Schema,CN=Configuration,DC=...
ms-DS-SiteName                              CN=ms-DS-SiteName,CN=Schema,CN=Configuration,DC=domain,D...
ms-DS-Top-Quota-Usage                       CN=ms-DS-Top-Quota-Usage,CN=Schema,CN=Configuration,DC=d...
ms-DS-User-Account-Control-Computed         CN=ms-DS-User-Account-Control-Computed,CN=Schema,CN=Conf...
ms-DS-User-Password-Expiry-Time-Computed    CN=ms-DS-User-Password-Expiry-Time-Computed,CN=Schema,CN...
Object-Classes                              CN=Object-Classes,CN=Schema,CN=Configuration,DC=domain,D...
Parent-GUID                                 CN=Parent-GUID,CN=Schema,CN=Configuration,DC=domain,DC=com
Possible-Inferiors                          CN=Possible-Inferiors,CN=Schema,CN=Configuration,DC=doma...
Primary-Group-Token                         CN=Primary-Group-Token,CN=Schema,CN=Configuration,DC=dom...
SD-Rights-Effective                         CN=SD-Rights-Effective,CN=Schema,CN=Configuration,DC=dom...
Structural-Object-Class                     CN=Structural-Object-Class,CN=Schema,CN=Configuration,DC...
SubSchemaSubEntry                           CN=SubSchemaSubEntry,CN=Schema,CN=Configuration,DC=domai...
Token-Groups                                CN=Token-Groups,CN=Schema,CN=Configuration,DC=domain,DC=com
Token-Groups-Global-And-Universal           CN=Token-Groups-Global-And-Universal,CN=Schema,CN=Config...
Token-Groups-No-GC-Acceptable               CN=Token-Groups-No-GC-Acceptable,CN=Schema,CN=Configurat...
```

[1]: http://msdn.microsoft.com/en-us/library/cc223368.aspx
[2]: http://msdn.microsoft.com/en-us/library/cc223369.aspx