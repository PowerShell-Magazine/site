---
title: '#PSTip New PowerShell drive (PSDrive)'
author: David Moravec
type: post
date: 2012-10-31T18:00:03+00:00
url: /2012/10/31/pstip-new-powershell-drive-psdrive/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
As PowerShell drives are really useful concept, many people use it a lot. I also created some PSDrives for parts of my system I am accessing frequently. Let’s see it in my **$profile**. My two most frequently used drives are:

```
PS> Get-PSDrive -Name D[or]* | Format-Table Name, Provider, Root -Auto
Name     Provider                             Root
----     --------                             ----
Download Microsoft.PowerShell.Core\FileSystem C:\Documents and Settings\Moravec\My Documents\Download
Dropbox  Microsoft.PowerShell.Core\FileSystem C:\Documents and Settings\Moravec\My Documents\Dropbox
```

My Dropbox folder&#8211;the obvious choice, and Download folder&#8211;that’s where I store all incoming files. You’ve already seen in one of the previous tips how to create a PowerShell drive. I can easily navigate to both of these drives:


    PS> cd download:
    PS Download:\> ls
    Directory: C:\Documents and Settings\Moravec\My Documents\Download
The Registry PowerShell provider doesn&#8217;t expose all of the Registry hives as PowerShell drives. By default, we only get two of them:

```
PS> cd download:
PS> Get-PSDrive -PSProvider Registry
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
HKCU                                   Registry      HKEY_CURRENT_USER
HKLM                                   Registry      HKEY_LOCAL_MACHINE
```

Sometimes I need also the others (especially as I frequently need to check/change/add something to HKEY_USERS). To create a new Registry PowerShell drive, you can use the following command:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; New-PSDrive -Name HKU  -PSProvider Registry –Root HKEY_USERS
</pre>

You can do the same with other Registry hives. After adding additional drives, you can access all common Registry parts easily:

```
PS> Get-PSDrive -PSProvider Registry
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
HKCC                                   Registry      HKEY_CURRENT_CONFIG
HKCR                                   Registry      HKEY_CLASSES_ROOT
HKCU                                   Registry      HKEY_CURRENT_USER
HKLM                                   Registry      HKEY_LOCAL_MACHINE
HKU                                    Registry      HKEY_USERS
```