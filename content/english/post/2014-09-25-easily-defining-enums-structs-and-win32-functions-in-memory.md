---
title: Easily Defining Enums, Structs, and Win32 Functions in Memory
author: Matt Graeber
type: post
date: 2014-09-25T16:00:46+00:00
url: /2014/09/25/easily-defining-enums-structs-and-win32-functions-in-memory/
categories:
  - How To
  - Security
  - InfoSec
tags:
  - How To
  - Security
  - InfoSec

---
In the past, I’ve spoken extensively on how to use reflection to define [enums, structs][1], and [Win32 functions][2] in PowerShell and the merits of doing so. [PowerSploit][3] is also heavily reliant upon these techniques in order to perform low-level operations while remaining entirely memory-resident.

Those familiar with using reflection to create .NET types on the fly (i.e. [Boe Prox][4], [Joe Bialek][5], [Will Schroeder][6], etc.) probably know how frustrating it is due to all the ceremony involved in having to control every nuance of the type being created. Fed up with the amount of work involved and considering I have a unique requirement to use reflection so often in my scripts, I needed a way to ease the process of defining in-memory enums, structs, and Win32 functions. I had the following design goals in mind:

  * Create intuitive wrapper functions around all the reflection code
  * Be able to declare enum, struct, and Win32 function definitions in as close to a “C-style” as PowerShell would allow – i.e. sometimes disregarding PowerShell best practices.
  * Structs should be aware of their size, eliminating the need to constantly call _[Runtime.InteropServices.Marshal]::SizeOf_ in my scripts.
  * Structs should allow for explicit conversion from an _IntPtr_, eliminating the need to constantly call _[Runtime.InteropServices.Marshal]::PtrToStructure_ in my scripts.
  * Maintain PowerShell 2.0 compatibility

### PSReflect

Introducing [PSReflect][7]&#8211;a series of helper functions designed to make defining in-memory enums, structs, and Win32 functions extremely easy.

PSReflect consists of the following helper functions:

<p style="padding-left: 30px;">
  <strong><em>New-InMemoryModule</em></strong> – Creates a host in-memory assembly and module
</p>

<p style="padding-left: 30px;">
  <em><strong>Add-Win32Type</strong></em> &#8211; Creates a .NET type for an unmanaged Win32 function
</p>

<p style="padding-left: 30px;">
  <strong><em>func</em></strong> – A helper function for Add-Win32Type that can be sued to eliminate typing when defining a large quantity of Win32 function definitions
</p>

<p style="padding-left: 30px;">
  <em><strong>enum</strong></em> &#8211; Creates an in-memory enumeration for use in your PowerShell session
</p>

<p style="padding-left: 30px;">
  <em><strong>struct</strong></em> &#8211; Creates an in-memory structure for use in your PowerShell session
</p>

<p style="padding-left: 30px;">
  <em><strong>field</strong></em> &#8211; A helper function for struct used to reduce typing while defining struct fields
</p>

### Starting Out

Before you can define anything, you must create an assembly and module that will host your enums, structs, and Win32 functions. It is really easy to create an in-memory assembly and module with the _New-InMemoryModule_ function.

<pre class="brush: powershell; title: ; notranslate" title="">$Mod = New-InMemoryModule
</pre>

Assemblies and modules require a name. If the ‘ModuleName’ parameter is not specified, it will create a GUID and use that as the name. With our in-memory module set up, we can now begin defining everything else.

### Enums

Defining enums couldn’t be easier. Here is an example enum definition:

<pre class="brush: powershell; title: ; notranslate" title="">$ImageDosSignature = enum $Mod PE.IMAGE_DOS_SIGNATURE UInt16 @{
   DOS_SIGNATURE =    0x5A4D
   OS2_SIGNATURE =    0x454E
   OS2_SIGNATURE_LE = 0x454C
   VXD_SIGNATURE =    0x454C
}
</pre>

Enums require the following components:

  * The in-memory module defined earlier
  * The full name (i.e. name and optionally, a namespace) of the enum
  * The underlying type of the enum elements
  * A hash table consisting of the enum elements

You can also optionally specify the following component:

  * Specify that the enum is a bitfield. This enables you to create and display an enum consisting of multiple elements binary OR’ed together.

### Structs

Structs are also now very easy to define, although the struct fields will require a little bit more information than a simple enum.

<pre class="brush: powershell; title: ; notranslate" title="">$ImageDosHeader = struct $Mod PE.IMAGE_DOS_HEADER @{
    e_magic =    field 0 $ImageDosSignature
    e_cblp =     field 1 UInt16
    e_cp =       field 2 UInt16
    e_crlc =     field 3 UInt16
    e_cparhdr =  field 4 UInt16
    e_minalloc = field 5 UInt16
    e_maxalloc = field 6 UInt16
    e_ss =       field 7 UInt16
    e_sp =       field 8 UInt16
    e_csum =     field 9 UInt16
    e_ip =       field 10 UInt16
    e_cs =       field 11 UInt16
    e_lfarlc =   field 12 UInt16
    e_ovno =     field 13 UInt16
    e_res =      field 14 UInt16[] -MarshalAs @('ByValArray', 4)
    e_oemid =    field 15 UInt16
    e_oeminfo =  field 16 UInt16
    e_res2 =     field 17 UInt16[] -MarshalAs @('ByValArray', 10)
    e_lfanew =   field 18 Int32
}
</pre>

Structs require the following components:

  * The in-memory module defined earlier
  * The full name (i.e. name and optionally, a namespace) of the struct
  * A hash table of the individual fields

You can also optionally specify the following components:

  * Packing size – i.e. the memory alignment of the fields
  * Explicit layout &#8211; Indicates that an explicit offset for each field will be specified

Each struct field is a hash table that requires the following components:

  * The order in which the field should be defined. Unfortunately, this cannot be omitted because PowerShell doesn’t not guarantee the order of defined elements in a hash table. Starting in PowerShell 3.0, you can prepend _[Ordered]_ to a hash table but that is not possible in PowerShell 2.0. The order in which fields are defined is essential, whereas with an enum, the order doesn’t matter.
  * The .NET type of the field

Optionally, struct fields may also contain the following components:

  * An explicit offset. If you indicate that your struct has an explicit layout, you must specify the offset of each field. An example of when you would want to use explicit offsets is when creating a union. PowerShell/.NET isn’t aware of the concept of a C union but you can define an equivalent &#8211; a struct with overlapping fields.
  * A _MarshalAs_ attribute. This is required if your struct contains a string or an array of objects. You have to use the _MarshalAs_ attribute to indicate the ‘unmanaged’ type and its size. For more information on this attribute, read this MSDN article (<http://msdn.microsoft.com/en-us/library/system.runtime.interopservices.unmanagedtype.aspx>).

Since each field of the struct is another hash table, I wrote the ‘field’ helper function to reduce the typing required.

Once your struct is defined, it behaves exactly the way you would expect only it has two additional features:

  * It comes with a built-in _GetSize_ method which returns the size of the struct in bytes.
  * It comes with a built-in explicit IntPtr conversion operator. This means that you can cast an _IntPtr_ to your struct type. This is very useful for me since many of my scripts contain calls to _[Runtime.InteropServices.Marshal]::PtrToStructure_. Having a built-in converter save a lot of typing.

### Win32 Functions

When using _Add-Win32Type_, it will be easiest to define your functions as an array of function declarations:

```
$FunctionDefinitions = @(
  (func kernel32 GetProcAddress ([IntPtr]) @([IntPtr], [String])),
  (func kernel32 GetModuleHandle ([Intptr]) @([String])),
  (func ntdll RtlGetCurrentPeb ([IntPtr]) @())
)

$Types = $FunctionDefinitions | Add-Win32Type -Module $Mod -Namespace 'Win32'
$Kernel32 = $Types['kernel32']
$Ntdll = $Types['ntdll']
```

Each function declaration requires the following:

  * The name of the DLL
  * The name of the DLL’s exported function
  * The return value of the function
  * An array of parameters that the function expects. If the function doesn’t have any parameters, just provide an empty array.

Optionally, you may also specify the following properties:

  * The native calling convention – The default is _stdcall_ but you may also specify _cdecl_ or _thiscall_. Unfortunately, .NET does not support the X86 fastcall calling convention.
  * The character set – If you want to explicitly call an ‘A’ or ‘W’ version of a function, you can specify either an ANSI or Unicode character set.

Once your function declarations are defined, you bake everything in with the _Add-Win32Type_ function at which point you provide your in-memory module defined earlier and optionally, a namespace for each type.

After the Win32 type are created, _Add-Win32Type_ returns a hash table of each type corresponding to each DLL name.

### Tying everything together

Now that we have all the requirements under our belt, we can start building some cool tools around this capability. Without further ado, I present to you a very basic PE DOS header parser – the “Hello, World” of working with structs in Windows. 😉

```
$Mod = New-InMemoryModule -ModuleName Win32 
$ImageDosSignature = enum $Mod PE.IMAGE_DOS_SIGNATURE UInt16 @{
    DOS_SIGNATURE =    0x5A4D
    OS2_SIGNATURE =    0x454E
    OS2_SIGNATURE_LE = 0x454C
    VXD_SIGNATURE =    0x454C
}

$ImageDosHeader = struct $Mod PE.IMAGE_DOS_HEADER @{
    e_magic =    field 0 $ImageDosSignature
    e_cblp =     field 1 UInt16
    e_cp =       field 2 UInt16
    e_crlc =     field 3 UInt16
    e_cparhdr =  field 4 UInt16
    e_minalloc = field 5 UInt16
    e_maxalloc = field 6 UInt16
    e_ss =       field 7 UInt16
    e_sp =       field 8 UInt16
    e_csum =     field 9 UInt16
    e_ip =       field 10 UInt16
    e_cs =       field 11 UInt16
    e_lfarlc =   field 12 UInt16
    e_ovno =     field 13 UInt16
    e_res =      field 14 UInt16[] -MarshalAs @('ByValArray', 4)
    e_oemid =    field 15 UInt16
    e_oeminfo =  field 16 UInt16
    e_res2 =     field 17 UInt16[] -MarshalAs @('ByValArray', 10)
    e_lfanew =   field 18 Int32
}

$FunctionDefinitions = @(
(func kernel32 GetModuleHandle ([Intptr]) @([String]))
)

$Type = $FunctionDefinitions | Add-Win32Type -Module $Mod -Namespace Win32
$Kernel32 = $Type['kernel32']

# Parse the DOS header of every loaded module in the PowerShell process

$PowerShellProc = Get-Process -Id $PID
$PowerShellProc.Modules | % {
    $Kernel32::GetModuleHandle($_.ModuleName) -as $ImageDosHeader
}
```

### PowerShell 2.0 Quirks

In every example provided, particular attention was paid to writing examples that specifically worked in PowerShell 2.0 and later. Specifically, all types must be saved to variables. In PowerShell 2.0, a bug exists where if you create a type using reflection, you cannot explicitly refer to its type despite it being properly loaded in your PowerShell session. For example, in the previous example, you wouldn&#8217;t be able to call _[Win32.kernel32]::GetModuleHandle_. Rather, in PowerShell 2.0, you have to save each type to a variable when it’s created. In PowerShell 3.0 and later, you can refer to your types explicitly using bracket syntax or via a variable.

If you’re in the unique position of needing to define types in memory, hopefully _PSReflect_ will help speed up your workflow.

[1]: http://www.exploit-monday.com/2012/07/structs-and-enums-using-reflection.html
[2]: http://blogs.technet.com/b/heyscriptingguy/archive/2013/06/27/use-powershell-to-interact-with-the-windows-api-part-3.aspx
[3]: https://github.com/mattifestation/PowerSploit
[4]: http://learn-powershell.net/2014/08/22/revisiting-get-filestamp-with-reflection/
[5]: http://clymb3r.wordpress.com/2013/05/26/implementing-remote-loadlibrary-and-remote-getprocaddress-using-powershell-and-assembly/
[6]: https://raw.githubusercontent.com/Veil-Framework/Veil-PowerView/master/powerview.ps1
[7]: https://github.com/mattifestation/PSReflect