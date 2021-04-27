---
title: '#PSTip View Policy Settings with RSOP WMI classes'
author: Shay Levy
type: post
date: 2013-04-02T16:15:05+00:00
url: /2013/04/02/pstip-view-policy-settings-with-rsop-wmi-classes/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires admin privileges (elevated shell)

To get Resultant Set of Policy (RSOP) data we usually _RSOP.mmc_ or the _gpresult_ command line tool. But we can also use WMI to retrieve the settings. Under the covers, the RSOP MMC snap-in calls WMI to create RSOP data. The data (snapshot) is created under temporary namespaces that are generated dynamically. For example, to get a list of user namespaces (partial result):

```
PS> Get-WmiObject -Namespace root\rsop\user -Class __NAMESPACE

__GENUS          : 2
__CLASS          : __NAMESPACE
__SUPERCLASS     : __SystemClass
__DYNASTY        : __SystemClass
__RELPATH        : __NAMESPACE.Name="S_1_5_21_1265342080_1232889825_337633078_123"
__PROPERTY_COUNT : 1
__DERIVATION     : {__SystemClass}
__SERVER         : SHAY
__NAMESPACE      : ROOT\rsop\user
__PATH           : \\SHAY\ROOT\rsop\user:__NAMESPACE.Name="S_1_5_21_1265342080_1232889825_337633078_123"
Name             : S_1_5_21_1265342080_1232889825_337633078_123
PSComputerName   : SHAY
(...)
```

Notice that the _Name_ property contains the SID of a user with underscores instead of dashes. With this in mind, we can grab the current user SID, replace the above characters, and get a list of RSOP classes and the policy information for the current user.

```
PS> $user = [Security.Principal.WindowsIdentity]::GetCurrent().User.Value -replace '-', '_'
PS> Get-WmiObject -Namespace root\rsop\user\$user -List RSOP*

   NameSpace: ROOT\rsop\user\S_1_5_21_1265342080_1232889825_337633078_123

Name                                Methods              Properties
----                                -------              ----------
RSOP_Session                        {}                   {creationTime, flags, id, SecurityGroups...}
RSOP_ExtensionStatus                {}                   {beginTime, displayName, endTime, error...}
RSOP_SOM                            {}                   {blocked, blocking, id, reason...}
RSOP_GPO                            {}                   {accessDenied, enabled, fileSystemPath, filterAllowed...}
RSOP_GPLink                         {}                   {appliedOrder, enabled, GPO, linkOrder...}
RSOP_ExtensionEventSourceLink       {}                   {eventSource, extensionStatus}
RSOP_ExtensionEventSource           {}                   {eventLogName, eventLogSource, id}
RSOP_PolicySetting                  {}                   {creationTime, GPOID, id, name...}
RSOP_RegistryPolicySetting          {}                   {command, creationTime, deleted, GPOID...}
RSOP_FolderRedirectionPolicySetting {}                   {creationTime, GPOID, grantType, id...}
RSOP_PushPrinterConnectionsPolic... {}                   {ConnectionType, creationTime, deleted, GPOID...}
RSOP_IEAKPolicySetting              {}                   {categories, channels, creationTime, customFavorites...}
RSOP_IERegistryPolicySetting        {}                   {command, creationTime, currentUser, deleted...}
(...)

Get-WmiObject -Namespace root\rsop\user\$user -Class RSOP_RegistryPolicySetting |
Format-List Name,registryKey,value*

Name        :
registryKey : Software\Policies\Microsoft\office\14.0\outlook\options\mail
value       : {}
valueName   :
valueType   : 0

Name        : TabProcGrowth
registryKey : Software\Policies\Microsoft\Internet Explorer\Main
value       : {50, 0, 0, 0}
valueName   : TabProcGrowth
valueType   : 1

Name        : **Command
registryKey : Software\Policies\Microsoft\Internet Explorer\Control Panel
value       : {42, 0, 42, 0...}
valueName   : **Command
valueType   : 0

Name        : ScreenSaveTimeOut
registryKey : Software\Policies\Microsoft\Windows\Control Panel\Desktop
value       : {57, 0, 48, 0...}
valueName   : ScreenSaveTimeOut
valueType   : 1

Name        :
registryKey : Software\Microsoft\Windows\CurrentVersion\Policies\Explorer
value       : {}
valueName   :
valueType   : 0

Name        : NoDriveTypeAutoRun
registryKey : Software\Microsoft\Windows\CurrentVersion\Policies\Explorer
value       : {255, 0, 0, 0}
valueName   : NoDriveTypeAutoRun
valueType   : 4
(...)
```

