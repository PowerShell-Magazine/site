---
title: '#PSTip Enhancements to implicit remoting in PowerShell 3.0'
author: Jan Egil Ring
type: post
date: 2013-03-22T18:00:59+00:00
url: /2013/03/22/pstip-enhancements-to-implicit-remoting-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Since PowerShell 2.0 we could use implicit remoting to import cmdlets from a remote PowerShell session into the local PowerShell session:

```
PS [1]: $session = New-PSSession -ComputerName dc01
PS [2]: Invoke-Command -Session $session -ScriptBlock {Import-Module ActiveDirectory}
PS [3]: Import-PSSession -Session $session -Module ActiveDirectory

ModuleType Name                                ExportedCommands
---------- ----                                ----------------
Script     tmp_mwxq3nlx.ko3                    {Add-ADComputerServiceAccount...
```

After running the commands in the above example we can run cmdlets from the _ActiveDirectory_ module, like _Get-ADUser_, without having the module installed on the local machine. The cmdlets we run from the imported module will run implicitly through the remote PowerShell session.

In PowerShell 3.0, we got more options in regards to implicit remoting. We now have a _–CimSession_ parameter on _Get-Module_, which can be used with _–ListAvailable_ to retrieve the modules available on the remote computer:

```
PS [4]: Get-Module -CimSession server01 -ListAvailable
ModuleType Name                                ExportedCommands
---------- ----                                ----------------
Manifest   AppLocker                           {Get-AppLockerFileInformation...
Manifest   Appx                                {Add-AppxPackage, Get-AppxPac...
Manifest   BestPractices                       {Get-BpaModel, Get-BpaResult,...
Manifest   BitsTransfer                        {Add-BitsFile, Complete-BitsT...
Manifest   BranchCache                         {Add-BCDataCacheExtension, Cl...
Manifest   CimCmdlets                          {Get-CimAssociatedInstance, G...
(...)
```

As we can see in the example above we did not need to first establish a session with the remote computer, we specified a computer name rather than a session object.

The same technique can be used with _Import-Module_ in order to implicitly import a module from a remote computer:

<pre class="brush: powershell; title: ; notranslate" title="">PS [5]: Import-Module -Name NetAdapter -CimSession hyperv01
</pre>

Now that the NetAdapter module is imported we can use the cmdlets available in the module, for example _Get-NetAdapter_:

```
PS [6]: Get-NetAdapter
Name                      InterfaceDescription                    ifIndex Status
----                      --------------------                    ------- -----
FailoverCluster_CSV_DS... HP NC551i Dual Port FlexFabric 10G...#3      14 Up
Hyper-V_VMSwitch_DS.He... HP NC551i Dual Port FlexFabric 10G...#2      13 Up
Hyper-V_LiveMigration_... HP NC551i Dual Port FlexFabric 10G...#4      15 Up
Hyper-V_VMSwitch_DS.iSCSI HP NC551i Dual Port FlexFabric 10G...#8      19 Up
Hyper-V_VMSwitch_DS.Admin HP NC551i Dual Port FlexFabric 10G...#7      18 Up
Hyper-V_VMSwitch_Trunk    HP NC551i Dual Port FlexFabric 10G...#5      16 Up
FailoverCluster_Heartb... HP NC551i Dual Port FlexFabric 10G...#6      17 Up
Server_Management_DS.A... HP NC551i Dual Port FlexFabric 10Gb ...      12 Up
```

The output in the example above lists the network adapters from the remote machine.

Note that only Cmdlet Definition XML (CDXML)-based modules can be imported this way. This is what happens if we try to import a non-CDXML-based module:

<pre class="brush: powershell; title: ; notranslate" title="">PS [7]: Import-Module -Name Hyper-V -CimSession hyperv01
Import-Module : The module Hyper-V cannot be imported over a CimSession.  Try u
sing the PSSession parameter of the Import-Module cmdlet.
At line:1 char:1
+ Import-Module -Name Hyper-V -CimSession hyperv01
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (Hyper-V:String) [Import-Module
   ], ArgumentException
    + FullyQualifiedErrorId : PsModuleOverCimSessionError,Microsoft.PowerShell
   .Commands.ImportModuleCommand
</pre>