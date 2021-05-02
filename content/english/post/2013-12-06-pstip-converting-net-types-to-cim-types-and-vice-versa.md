---
title: '#PSTip Converting .NET types to CIM types and vice versa'
author: Shay Levy
type: post
date: 2013-12-06T19:00:18+00:00
url: /2013/12/06/pstip-converting-net-types-to-cim-types-and-vice-versa/
categories:
  - WMI
  - Tips and Tricks
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 3.0 or above.

If you get a list of CIM class properties you&#8217;ll notice the _CimType_ property.

```
PS> $class = Get-CimClass -ClassName Win32_Process
$class.CimClassProperties | Format-Table Name,CimType

Name                        CimType
----                        -------
Caption                      String
Description                  String
InstallDate                DateTime
Name                         String
Status                       String
CreationClassName            String
CreationDate               DateTime
CSCreationClassName          String
CSName                       String
ExecutionState               UInt16
Handle                       String
KernelModeTime               UInt64
OSCreationClassName          String
OSName                       String
Priority                     UInt32
TerminationDate            DateTime
UserModeTime                 UInt64
WorkingSetSize               UInt64
CommandLine                  String
ExecutablePath               String
HandleCount                  UInt32
(...)
```

The _CimType_ property specifies a CIM type, such as integer, string, or datetime. The _CimConverter_ class allows us to convert CIM types to the equivalent .NET type using the _GetDotNetType_ static method. The following snippet lists the values of the _CimType_ enum and creates a table of the CIM type and its .NET equivalent.

```
[Microsoft.Management.Infrastructure.CimType].GetEnumNames() | ForEach-Object {
    [PSCustomObject]@{
        CimType = $_
        NetType = [Microsoft.Management.Infrastructure.CimConverter]::GetDotNetType($_)
    }
}

CimType        NetType
-------        -------
Unknown
Boolean        System.Boolean
UInt8          System.Byte
SInt8          System.SByte
UInt16         System.UInt16
SInt16         System.Int16
UInt32         System.UInt32
SInt32         System.Int32
UInt64         System.UInt64
SInt64         System.Int64
Real32         System.Single
Real64         System.Double
Char16         System.Char
DateTime
String         System.String
Reference      Microsoft.Management.Infrastructure.CimInstance
Instance       Microsoft.Management.Infrastructure.CimInstance
BooleanArray   System.Boolean[]
UInt8Array     System.Byte[]
SInt8Array     System.SByte[]
UInt16Array    System.UInt16[]
SInt16Array    System.Int64[]
UInt32Array    System.UInt32[]
SInt32Array    System.Int32[]
UInt64Array    System.UInt64[]
SInt64Array    System.Int64[]
Real32Array    System.Single[]
Real64Array    System.Double[]
Char16Array    System.Char[]
DateTimeArray
StringArray    System.String[]
ReferenceArray Microsoft.Management.Infrastructure.CimInstance[]
InstanceArray  Microsoft.Management.Infrastructure.CimInstance[]
```

The _GetCimType_ static method allows us to go the other direction and convert a .NET type to CIM type. A value of _&#8216;Unknown&#8217;_ is returned if the mapping is ambiguous.

<pre class="brush: powershell; title: ; notranslate" title="">[Microsoft.Management.Infrastructure.CimConverter]::GetCimType('int')
SInt32
</pre>