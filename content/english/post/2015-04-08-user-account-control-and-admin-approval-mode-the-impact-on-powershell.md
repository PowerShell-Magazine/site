---
title: User Account Control and Admin Approval Mode – The impact on PowerShell
author: Jaap Brasser
type: post
date: 2015-04-08T16:00:00+00:00
url: /2015/04/08/user-account-control-and-admin-approval-mode-the-impact-on-powershell/
views:
  - 19216
post_views_count:
  - 4751
categories:
  - How To
tags:
  - How To

---
In modern days Windows operating systems, User Account Control (UAC) is enabled by default. There are some considerations when using PowerShell and running a script in elevated or non-elevated context. This and some of the issues that can occur because of UAC will be addressed in this article.

UAC is configured in Admin Approval Mode by default. In this Mode, when an administrator logs on to a system, two tokens are assigned to the user &#8212; a normal full control token and a special filtered access token. This helps prevent an administrator from making an accidental change that might have system wide consequences or prevent the accidental installation of a malicious program. The filtered access token is created during the logon process and is used to start explorer.exe process. Any application that is started from this moment on will by default be launched using this filtered access token.

There are some slight differences to the configuration of Admin Approval Mode on client systems and server systems. The most notable one is that on Windows Server the default administrator account is enabled and is the only account for which Admin Approval Mode is disabled by default. This allows for an easy bypass on Windows Server systems. On Windows Client systems, the administrator account is disabled by default. However, once this administrator account is enabled, Admin Approval Mode is disabled by default on client systems too.

To show how Admin Approval Mode is relevant in PowerShell context, the next demonstration will show what happens when a drive mapping is created in both an elevated and non-elevated PowerShell console. The impact that this has for PowerShell can be demonstrated by opening two consoles, one regular console using the filtered access token and one elevated console, started using Run as Administrator. In both consoles, a drive mapping will be created by typing the following commands:

```powershell
New-PSDrive -Name Y -PSProvider FileSystem -Root \localhost\c$ -Persist
New-PSDrive -Name Z -PSProvider FileSystem -Root \localhost\c$ -Persist
```

To verify the results of this command in each console, the Get-PSDrive cmdlet can be executed to verify which mapped drives are visible in the current context:

```powershell
Get-PSDrive –PSProvider FileSystem
```

![](/images/UAC_PS_01.png)

It is clearly visible that the mapped drives for either token are unavailable in the other sessions. This can also be seen by opening File Explorer, which is also running using the filtered access token:

![](/images/UAC_PS_02.png)

As expected, only the mapped drive created in the non-elevated PowerShell console is visible from File Explorer. Because this is part of the UAC security infrastructure, this problem is not limited to PowerShell but can also be observed when mapping drives in command prompt. This article will limit to the implications that this security setting has on PowerShell, specifically when mapping network drives. The take away here is that when mapping network drives the correct security context should be used.

There are workarounds available for this issue, one method is to disable Admin Approval Mode for all administrators. Because this is a registry setting with significant impact on how User Account Control works a restart is required. Disabling Admin Approval Mode has a distinct disadvantage, it will cause Modern Apps to stop working. The following error message will be displayed:

![](/images/UAC_PS_03.png)

If this is not a concern, the following PowerShell line can be executed to disable Admin Approval mode all together:

```powershell
New-ItemProperty -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System -Name EnableLUA -Value 0 -Force
```

To enable Admin Approval Mode, the same command can be executed, changing the value from zero to one.

The preferred workaround is to Enable Linked Connections registry setting. When this setting is enabled, drive mappings are mirrored between the filtered access token and full control token. Because the drive mappings are being mirrored, the drives that are mapped in a non-elevated console will be visible in an elevated console and vice-versa. A restart is usually required to make this registry setting active. This setting can be enabled by creating the following registry key using PowerShell:

```powershell
New-ItemProperty -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System -Name EnableLinkedConnections -Value 1 –Force
```

To verify the results of this registry setting, the same commands that were executed at the beginning of this article will be repeated in a non-elevated console and an elevated console.

The results are as follows:

![](/images/UAC_PS_04.png)

And when File Explorer is opened, both mapped network drives are available:

![](/images/UAC_PS_05.png)

When working with modern operating systems and PowerShell, it is important to keep in mind the challenges that are introduced by UAC. While UAC can be completely disabled, this can have undesirable side-effects such as Modern Apps not being functional and a less secure computing environment. If it is important to have the same mapped drives in non-elevated PowerShell consoles as well as the elevated PowerShell consoles, the EnableLinkedConnections registry can be set to facilitate this.