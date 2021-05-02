---
title: 'PSTip: Connect to Azure Virtual Machines without being prompted for credentials'
author: Jaap Brasser
type: post
date: 2015-04-16T16:00:01+00:00
url: /2015/04/16/pstip-connect-to-azure-virtual-machines-without-being-prompted-for-credentials/
views:
  - 10807
post_views_count:
  - 1671
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
When working with Azure it can be useful to quickly connect to a number of Azure VMs. Unfortunately this might result in having to enter your password multiple times. To circumvent this problem I wrote the Connect-Mstsc function which accepts a user name and password as parameters. The function stores the credentials in the local credential store and because of this the credentials do not have to be separately entered when connecting via RDP to the remote system. Although the Get-AzureRemoteDesktopFile is a useful cmdlet that creates an .RDP file for Remote Desktop connection, it does prompt for the credentials.

For example, to connect to an Azure Virtual Machine using its hostname and port number:

```powershell
$Cred = Get-Credential
Connect-Mstsc –ComputerName cloudservice.cloudapp.net:58142 –Credential $Cred
```

This also works in combination with the Azure PowerShell, if a RDP session needs to be created with all of the systems that have RDP configured the following code can be used. It filters based on the AzureEndpoint which has port 3389 configured:

```powershell
$Cred = Get-Credential
Get-AzureVM | Get-AzureEndPoint | Where-Object {$_.LocalPort -eq 3389} | ForEach-Object {
    Connect-Mstsc -ComputerName ($_.Vip,$_.Port -join ':') -Credential $Cred
} 
```


The Connect-Mstsc function is available for download in the TechNet Script Gallery: [Connect-Mstsc][1]

[1]: https://gallery.technet.microsoft.com/scriptcenter/Connect-Mstsc-Open-RDP-2064b10b