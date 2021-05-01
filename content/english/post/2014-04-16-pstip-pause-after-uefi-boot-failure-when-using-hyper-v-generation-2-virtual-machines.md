---
title: '#PSTip Pause after UEFI boot failure when using Hyper-V Generation 2 virtual machines'
author: Jan Egil Ring
type: post
date: 2014-04-16T18:00:44+00:00
url: /2014/04/16/pstip-pause-after-uefi-boot-failure-when-using-hyper-v-generation-2-virtual-machines/
categories:
  - Hyper-V
  - Tips and Tricks
  - WMI
tags:
  - Hyper-V
  - Tips and Tricks
  - WMI

---
Hyper-V in Windows 8.1 and Windows Server 2012 R2 introduced Generation 2 virtual machines, which provides new feature and better performance. One of the new features is UEFI, a replacement of the traditional BIOS. When troubleshooting boot problems on Generation 2 virtual machines, for example as part of a new image build, one of the issues is that the boot process is so fast it is hard to follow what is happening.

For advanced users, there is a CIM property in the _[Msvm_VirtualSystemSettingData][1]_ class called _PauseAfterBootFailure_:

```
Get-CimClass -Namespace 'Root\Virtualization\V2' -ClassName Msvm_VirtualSystemSettingData |
Select-Object -ExpandProperty CimClassProperties | where name -eq "PauseAfterBootFailure"

Name               : PauseAfterBootFailure
Value              :
CimType            : Boolean
Flags              : Property, NullValue
Qualifiers         : {read, write}
ReferenceClassName :
```

This property is by default set to $false. During troubleshooting it might be useful to enable this setting. Normally we would expect to accomplish this is by leveraging the CIM cmdlets:

<pre class="brush: powershell; title: ; notranslate" title="">Get-CimInstance -Namespace Root\Virtualization\V2 -ClassName Msvm_VirtualSystemSettingData -Filter "ElementName = 'Demo-VM'" |
Set-CimInstance -Property @{PauseAfterBootFailure=$true} –PassThru
</pre>

Alternatively, by using _Get-WmiObject_:

<pre class="brush: powershell; title: ; notranslate" title="">$vm = Get-WmiObject -Class Msvm_VirtualSystemSettingData -Namespace Root\Virtualization\V2 -Filter "ElementName = 'Demo-VM'"
$vm.PauseAfterBootFailure = $true
$vm.Put()
</pre>

Although these commands does not produce any error messages, the property remains unchanged.

The reason for this is that the original WMI APIs did not provide a way to pass a WMI instance as the argument to a method directly; to pass an object to a method, you would serialize it into an embedded instance, a string representation of the object in XML form (using the _ManagementObject.GetText_ method).

The Hyper-V WMI provider continues to expect any input objects in embedded-instance format, but the CimInstance objects retrieved by the CIM cmdlets do not currently provide a convenience method for serializing an object into an embedded instance.

```
#Virtual System Management Service
$VSMS = Get-CimInstance -Namespace root/virtualization/v2 -Class Msvm_VirtualSystemManagementService

#Virtual Machine
$VM = Get-CimInstance -Namespace root/virtualization/v2  -Class Msvm_ComputerSystem -Filter "ElementName='Demo-VM'"

#Setting Data
$SD = $vm | Get-CimAssociatedInstance -ResultClassName Msvm_VirtualSystemSettingData -Association Msvm_SettingsDefineState

#Update boot option
$SD.PauseAfterBootFailure = $True

#Create embedded instance
$cimSerializer = [Microsoft.Management.Infrastructure.Serialization.CimSerializer]::Create()
$serializedInstance = $cimSerializer.Serialize($SD, [Microsoft.Management.Infrastructure.Serialization.InstanceSerializationOptions]::None)
$embeddedInstanceString = [System.Text.Encoding]::Unicode.GetString($serializedInstance)

#Modify the system settings
Invoke-CimMethod -CimInstance $VSMS -MethodName ModifySystemSettings @{SystemSettings = $embeddedInstanceString}
```

When the PauseAfterBootFailure property is set to $true, the specified virtual machine will ask the user to press a key when a boot attempt failed:

![](/images/Hyper-V_PauseAfterBootFailure-300x177.png)

The same technique can be used to modify other writable properties in the [Msvm_VirtualSystemSettingData][1] class.

Thanks to Brian Young at Microsoft for his assistance explaining this behavior.

[1]: http://msdn.microsoft.com/en-us/library/hh850257(v=vs.85).aspx