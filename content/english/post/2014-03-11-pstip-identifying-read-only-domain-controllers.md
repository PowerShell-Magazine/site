---
title: '#PSTip Identifying read-only domain controllers'
author: Shay Levy
type: post
date: 2014-03-11T18:00:59+00:00
url: /2014/03/11/pstip-identifying-read-only-domain-controllers/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

<span style="line-height: 1.5em;">When you get a list of domain controllers using the AD module, one of the </span>properties<span style="line-height: 1.5em;"> each DC has is the <em>IsReadOnly</em> property. </span>When _IsReadOnly_ is set to _$true_, the domain controller is a read-only domain controller.

<pre class="brush: powershell; title: ; notranslate" title="">Import-Module ActiveDirectory
Get-ADDomainController -Filter * | Select-Object Name,IsReadOnly
</pre>

One way to get RODCs is to filter the above result using the _Where-Object_ cmdlet:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ADDomainController -Filter * | Where-Object {$_.IsReadOnly -eq $true}
</pre>

But there&#8217;s a better and efficient way than that. Using the _Filter_ parameter you filter the objects on the server and get back just the ones that meet the filter criteria whereas piping to _Where-Object_ will get all objects and only then filtering will occur.

<pre class="brush: powershell; title: ; notranslate" title="">Get-ADDomainController -Filter {IsReadOnly -eq $true}
</pre>

Without the AD module you can search for read-only domain controllers by querying their _primaryGroupID_ attribute (primary group). RODCs will have a value of 521 which is the _&#8220;Read-only Domain Controllers&#8221;_ built-in AD group (writable DCs have the _primaryGroupID_ set to 516, which is the _&#8220;Domain Controllers&#8221;_ group).

```
([adsisearcher]'(primaryGroupID=521)').FindAll()
```