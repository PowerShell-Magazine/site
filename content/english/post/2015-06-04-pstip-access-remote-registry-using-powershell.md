---
title: '#PSTip Access remote registry using PowerShell'
author: Jaap Brasser
type: post
date: 2015-06-04T21:28:44+00:00
url: /2015/06/04/pstip-access-remote-registry-using-powershell/
views:
  - 27658
post_views_count:
  - 5745
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Using the Microsoft.Win32.Registry class it is possible to access both&#8211;the local registry and, more importantly, the registry of a remote system. Using the PowerShell cmdlet this is unfortunately not possible. Ravikanth posted a similar [tip][1] in PowerShell Magazine in which he uses it to retrieve SQL instance names using remote registry.

The following code will retrieve a list of sub keys from the HKLM:\Software path in the registry on Server1 computer:

```powershell
$Reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine,Server1) 

$RegSubKey = $Reg.OpenSubKey("System\CurrentControlSet")
$Values = $RegSubKey.GetSubKeyNames()
```

Using this information it is possible to extract a value from the subkey as well. The following code sample will extract the path parameter from the first subkey in the Software container.

```powershell
$RegPath = Join-Path 'Software' $Values[1]
$Reg.OpenSubKey($RegPath).GetValue('Path') 
```

For more information about this class and the available methods please refer to the MSDN article:

[RegistryKey Class][2]

[1]: http://104.131.21.239/2013/08/06/pstip-retrieve-all-sql-instance-names-on-local-and-remote-computers/
[2]: https://msdn.microsoft.com/en-us/library/microsoft.win32.registrykey