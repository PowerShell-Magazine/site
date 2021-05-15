---
title: Client and patch management using the UpdateServices module
author: Boe Prox
type: post
date: 2012-11-07T13:19:22+00:00
url: /2012/11/07/client-and-patch-management-using-the-updateservices-module/
categories:
  - How To
  - Module Spotlight
tags:
  - How To
  - Modules

---
With the release of Windows Server 2012 into the world, automating and managing Windows Server Update Services (WSUS) with PowerShell has become a little easier with the inclusion of the UpdateServices module. With the UpdateServices module, you get 12 cmdlets available to handle some of the more basic administration with WSUS that are listed in the table below.

| **Add-WsusComputer**       | **Get-WsusServer**                |
| -------------------------- | --------------------------------- |
| **Approve-WsusUpdate**     | **Get-WsusUpdate**                |
| **Deny-WsusUpdate**        | **Invoke-WsusServerCleanup**      |
| **Get-WsusClassification** | **Set-WsusClassification**        |
| **Get-WsusComputer**       | **Set-WsusProduct**               |
| **Get-WsusProduct**        | **Set-WsusServerSynchronization** |

For this article, we will be using 5 of the cmdlets to manage the clients and updates on the WSUS server: _Add-WsusComputer, Get-WsusUpdate, Deny-WsusUpdate, Get-WsusUpdate, and Get-WsusComputer_.

Before moving forward with working with the UpdateServices module, you need to make sure that it is installed. Finding this can be done by running the following command.

```powershell
Get-Module -ListAvailable –Name UpdateServices
```

![](/images/BoeWsus1.png)

If you see the module, then you are ready to go, but if not, then the below steps will help you out to install the module on your system.

### Enable UpdateServices module using the UI

Open up Server Manager and then click Add Roles and Features to bring up wizard. Click next with the default selection of Role-based or feature-based installation. Click next at the next two windows until you are at the Features selection. Expand Remote Server Administration Tools, and then select Windows Server Update Services Tools. This will install the API, PowerShell module, and the Management Console for WSUS.

![](/images/BoeWsus2.png)

### Enable UpdateServices module using PowerShell

Open up PowerShell and run the following command to enable WSUS.

```powershell
Install-WindowsFeature -Name UpdateServices-RSAT
```

A reboot will not be required once this has completed. When you have installed the UpdateServices module, you are ready to go with managing WSUS!

#### Client management

Finding all of your connected clients is as simple as running _Get-WsusComputer_. This command will show you all of your clients:

```powershell
Get-WsusComputer
```

![](/images/BoeWsus3.png)

The _Get-WsusComputer_ cmdlet contains some great parameters to help you filter exactly what you are looking for when searching for clients.

For instance, if you want to clean out clients that haven’t reported in 30 days, the following code will locate and remove the clients. Note that there is not a native cmdlet for removing a client from WSUS nor is it publicly available in the _Microsoft.UpdateServices.Commands.WsusComputer_ object that _Get-WsusComputer_ returns. We can work around this by using the WSUS API via the _Get-WsusServer_ cmdlet and then use each computer name to call the _GetComputerTargetByName()_ method followed by the _Delete()_ method in the _Microsoft.UpdateServices.Internal.BaseApi.ComputerTarget_ object that is available. I chose this hybrid approach because we can easily filter using the built-in cmdlet to get the list of computers and then run each of those through using the API to remove them from the server.

```powershell
#Used for the WSUS APIs
$wsus = Get-WsusServer
Get-WsusComputer -FromLastReportedStatusTime (Get-Date).AddDays(-30) | ForEach-Object {
	Write-Verbose ("Removing {0} from WSUS" -f $_.FullDomainName)
        $wsus.GetComputerTargetByName($_.Name).Delete()
}
```


Adding a client in WSUS to an existing Target Group is made simple using _Add-WsusComputer_ and supplying a parameter for the _TargetGroupName_.

```powershell
Get-WsusComputer -NameIncludes Boe-PC |
Add-WsusComputer -TargetGroupName "Windows 2012" –Verbose
```


![](/images/BoeWsus4.png)

### Update management

Similar to working with the clients, you can find all of the updates currently listed on the WSUS server by using _Get-WsusUpdate_ A word of caution, using _Get-WsusUpdate_ without any parameters will list all of the updates on the server which can take a bit of time.

```powershell
Get-WsusUpdate | Select-Object -First 10
```

![](/images/BoeWsus5.png)

My personal favorite parameter on this cmdlet is _–Status_ parameter which can easily filter for updates that are needed by the clients. The _–Status_ parameter takes the type of _Microsoft.UpdateServices.Commands.WsusUpdateInstallationState_. One way to find what the acceptable value is by using the following command:

```powershell
[Enum]::GetNames(Microsoft.UpdateServices.Commands.WsusUpdateInstallationState)
NoStatus
InstalledOrNotApplicable
InstalledOrNotApplicableOrNoStatus
Failed
Needed
FailedOrNeeded
Any
```

With this knowledge, we can run the following command to get all updates that are needed and have not been approved yet on the WSUS server.

```powershell
Get-WsusUpdate -Status Needed -Approval Unapproved
```

![](/images/BoeWsus6.png)

These are just a few of the updates that are needed by the clients that will need to be approved so they can be compliant with patches.

Now that we know how to get the required updates from the WSUS server, now it is time to start the approval process using _Approve-WsusUpdate_. The best approach to using this cmdlet is to pipe the output of _Get-WsusUpdate_ into the _Approve-WsusUpdate_ cmdlet followed by the _TargetGroup_ and _UpdateApprovalAction_ to complete the approval process.

```powershell
Get-WsusUpdate -Status Needed -Approval Unapproved |
Approve-WsusUpdate -Action Install -TargetGroupName "All Computers" –Verbose
```


On the flip-side, there are times when an update is not needed and was approved on accident or is just no longer needed by the clients. For this case, _Deny-WsusUpdate_ is available to use to quickly decline the updates.

```powershell
Get-WsusUpdate -Approval Approved -Status NoStatus |
Deny-WsusUpdate –Verbose
```


So there you go! Using these cmdlets can allow you to quickly automate your patch and client management duties with PowerShell!