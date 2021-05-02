---
title: '#PSTip Deleting expired certificates from the personal certificate store'
author: Josh Miller
type: post
date: 2013-06-19T18:00:17+00:00
url: /2013/06/19/pstip-deleting-expired-certificates-from-the-personal-certificate-store/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
We use a lot of certificates for website authentication, and they expire each year.  Some people end up with a collection of expired certificates. That makes it hard to find the right one when presented with the list of certificates in Internet Explorer.

To work with the certificates we use the X.509 Certificate Provider (_Microsoft.PowerShell.Security\Certificate_).  This provider in PowerShell 2.0 requires jumping through a few manual hoops to clean up the environment.  We need to work with the [System.Security.Cryptography.X509Certificates.X509Store][1] object associated with the store and grab that with Get-Item:

<pre class="brush: powershell; title: ; notranslate" title="">$myCerts = Get-Item Cert:\CurrentUser\My
</pre>
To delete the certificates we have to open the _$myCerts_ X.509 Store object passing an _[OpenFlag][2]s_ enumeration.  The _Open()_ method can create a new store or set what access is given to the given store based on the _[OpenFlag][2]s_.  By default the store is read-only and you cannot remove the certificate without opening it.

| **Member name**  | **Description**                                              |
| ---------------- | ------------------------------------------------------------ |
| IncludeArchived  | Open the X.509 certificate store and include archived certificates. |
| MaxAllowed       | Open the X.509 certificate store for the highest access allowed. |
| OpenExistingOnly | Opens only existing stores; if no store exists, the Open method will not create a new store. |
| ReadOnly         | Open the X.509 certificate store for reading only.           |
| ReadWrite        | Open the X.509 certificate store for both reading and writing. |

<pre class="brush: powershell; title: ; notranslate" title="">$myCerts.Open([System.Security.Cryptography.X509Certificates.OpenFlags]::ReadWrite)
</pre>

To get the list of expired certificates we need to filter the child items that are not valid after yesterday.  _$myCerts_ is already pointing to the path we need, so we can use it as a reference rather than repeating a hard coded string.

<pre class="brush: powershell; title: ; notranslate" title="">$today = Get-Date
$ExpiredList = Get-ChildItem $myCerts.PSPath | Where-Object { $_.NotAfter -lt $today }
</pre>

There is a reason we set _$toda_y before the pipeline,  _Get-Date_ does work, we don’t want to call if for every iteration of the _Where-Object_ cmdlet as we don’t need the refined difference between each call.

Remove each certificate in the X.509 certificate store that was returned from our query:

```
ForEach ($Cert in $ExpiredList) {
	$myCerts.Remove($Cert)
}

$myCerts.Close() # We opened it, so we need to close it.
```

In PowerShell 3.0 the entire thing can be done:

```
$today = Get-Date
Get-ChildItem Cert:\CurrentUser\My |
Where-Object NotAfter -lt $today |
Remove-Item

#or

Get-ChildItem Cert:\CurrentUser\My |
ForEach-Object -begin { $now = get-date } -process { if ($PSItem.NotAfter -lt $now ) { $PSItem } } |
Remove-Item
```

[1]: http://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509store.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.openflags.aspx