---
title: '#PSTip Excluding WMI system properties'
author: Shay Levy
type: post
date: 2012-12-09T10:57:26+00:00
url: /2012/12/09/pstip-excluding-wmi-system-properties/
post_views_count:
  - 1960
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Whenever you use Windows Management Instrumentation (WMI), the result always comes back with a list of system properties. WMI system properties are associated with all classes and instances of classes, and begin with a double underscore.

```
PS> Get-WmiObject Win32_Processor
__GENUS                     : 2
__CLASS                     : Win32_Processor
__SUPERCLASS                : CIM_Processor
__DYNASTY                   : CIM_ManagedSystemElement
__RELPATH                   : Win32_Processor.DeviceID="CPU0"
__PROPERTY_COUNT            : 48
__DERIVATION                : {CIM_Processor, CIM_LogicalDevice, CIM_LogicalElement, CIM_ManagedSystemElement}
__SERVER                    : LOKI
__NAMESPACE                 : root\cimv2
__PATH                      : \\LOKI\root\cimv2:Win32_Processor.DeviceID="CPU0"
AddressWidth                : 64
Architecture                : 9
Availability                : 3
Caption                     : Intel64 Family 6 Model 44 Stepping 2
ConfigManagerErrorCode      :
ConfigManagerUserConfig     :
CpuStatus                   : 1
CreationClassName           : Win32_Processor
CurrentClockSpeed           : 2800
CurrentVoltage              : 33
DataWidth                   : 64
Description                 : Intel64 Family 6 Model 44 Stepping 2
(...)
```

Sometimes you&#8217;d want to remove system properties from the output, when constructing a report for example,  and you might attempt to do that with the _ExcludeProperty_ parameter of the _Select-Object_ cmdlet, but as you can see it doesn&#8217;t seem to work, all system properties are still in place.

```
PS> Get-WmiObject Win32_Processor | Select-Object -ExcludeProperty __*
__GENUS                     : 2
__CLASS                     : Win32_Processor
__SUPERCLASS                : CIM_Processor
__DYNASTY                   : CIM_ManagedSystemElement
__RELPATH                   : Win32_Processor.DeviceID="CPU0"
__PROPERTY_COUNT            : 48
__DERIVATION                : {CIM_Processor, CIM_LogicalDevice, CIM_LogicalElement, CIM_ManagedSystemElement}
__SERVER                    : LOKI
__NAMESPACE                 : root\cimv2
__PATH                      : \\LOKI\root\cimv2:Win32_Processor.DeviceID="CPU0"
AddressWidth                : 64
Architecture                : 9
(...)
```

In order to exclude them you must first include all properties. The _ExcludeProperty_ parameter is effective only when the command also includes the _Property_ parameter.

```
PS> Get-WmiObject Win32_Processor | Select-Object -Property * -ExcludeProperty __*

AddressWidth                : 64
Architecture                : 9
Availability                : 3
Caption                     : Intel64 Family 6 Model 44 Stepping 2
ConfigManagerErrorCode      :
ConfigManagerUserConfig     :
CpuStatus                   : 1
CreationClassName           : Win32_Processor
CurrentClockSpeed           : 2800
CurrentVoltage              : 33
DataWidth                   : 64
Description                 : Intel64 Family 6 Model 44 Stepping 2
(...)
```