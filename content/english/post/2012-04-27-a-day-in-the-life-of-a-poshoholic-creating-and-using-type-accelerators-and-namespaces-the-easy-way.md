---
title: 'A day in the life of a Poshoholic: Creating and using type accelerators and namespaces the easy way'
author: Kirk Munro
type: post
date: 2012-04-27T18:00:20+00:00
url: /2012/04/27/a-day-in-the-life-of-a-poshoholic-creating-and-using-type-accelerators-and-namespaces-the-easy-way/
views:
  - 15872
post_views_count:
  - 2125
categories:
  - How To
tags:
  - How To

---
You never know where you’re going to find inspiration.  As a PowerShell MVP and a confessed Poshoholic, I tend to travel to conferences a fair bit.  I go out of my way to attend those conferences and participate in discussions with other attendees so that I can hear what they have to say about PowerShell.  PowerShell is great, but it’s far from perfect, and these conversations help me identify ways I can contribute to helping make the overall experience better.  As much as these intentional interactions are great, it’s often even more useful to have the opportunity to be a fly on the wall, listening to (ok, eavesdropping on) discussions about PowerShell that are taking place in unexpected places, looking for seeds of inspiration that can grow into something that makes PowerShell even better.

At TechEd North America 2011 I had one such opportunity present itself. While I was working in the speakers lounge on my own presentation materials, I overheard a couple of C# developers talking and one of them was saying that they hate PowerShell.  My interest was piqued, so I focused less on my own work and more on what they were saying.  If I heard correctly (it can be difficult to catch what’s being said when you’re trying to inconspicuously eavesdrop on a conversation), the developer who was complaining had two major issues.  I’ll touch on the first issue in this article, and save the other one for a follow-up article.

### PowerShell does not have a using keyword

The first issue they pointed out was that they didn’t like having to type in so many full .NET type names when they typecast or when they create strongly typed objects.  In C#, developers can work with shorter type names by invoking the using statement, allowing them to control exactly how much of a type name is required in order for the compiler to be able to recognize the type.  For example, in C# if you were to write some code to work with a bunch of Windows Forms objects, you could do this:

```c#
using System.Windows.Forms;

DialogResult result = MessageBox.Show(…);
```

In this code snippet, both DialogResult and MessageBox can be used in the program without having to qualify them with their namespace because the using statement indicates that the namespace is automatically identified for objects that are defined within it.  This is quite convenient and it can save a lot of typing in the long run.

PowerShell doesn’t have a using statement however, at least not in PowerShell 2.0.  The using keyword is reserved in PowerShell 2.0, but there is no statement or command in place that defines what that keyword will do.  Using PowerShell 2.0 out of the box, if you wanted to create a script similar to the C# code that is shown above, you would have to do this:

```powershell
[System.Windows.Forms.DialogResult]$result =
```

Since there is no using statement support, with long type names like these your PowerShell scripts may quickly require a lot more typing if you are interacting with .NET object types directly in those scripts.  As someone who used to write C# code, this can definitely be disappointing.  Fortunately though, it doesn’t have to be this way.

### Type Accelerators

Windows PowerShell 2.0 comes with built-in support for 31 type accelerators.  Type accelerators are simply mappings between shorthand type names and the full object types that they represent.  They exist to allow you to use a shorthand type name instead of a full type name.  For example, you can create a switch parameter in a function by simply using the [switch] type name.  Behind the scenes, PowerShell knows that [switch] is really [System.Management.Automation.SwitchParameter], but [switch] is much, much easier to type and use.

Retrieving a list of type accelerators that exist in PowerShell is not easy.  There are no cmdlets and no published interface that allows you to do this out of the box.  Fortunately though there are some examples on various blog posts that identify a hidden, private type that can be used to get this list.  That type is System.Management.Automation.TypeAccelerators, and it can be accessed in PowerShell like this:

```powershell
[System.Type]$typeAcceleratorsType =
[System.Management.Automation.PSObject].Assembly.GetType(
'System.Management.Automation.TypeAccelerators',
$true,
$true
)
```


It is possible to call GetType without passing in the second and third Boolean parameters, however if you do that you must make sure the type name you use has the proper case.  The second parameter indicates you want the method to throw an exception if there is an error, and the third parameter indicates you want to perform a case insensitive lookup.  Since PowerShell itself is case insensitive, I tend to use all three parameters here so that I don’t have to worry about case sensitivity.

Once you have access to that type, management of type accelerators is pretty straightforward and is done by using three static methods of that type.  You can see the current list of accelerators by invoking $typeAcceleratorsType::Get.  You can add your own type accelerator by invoking $typeAcceleratorsType::Add and passing in the string you want to use as an accelerator ad the type that you want it to be associated with (note: in PowerShell 3.0 CTP 1, this method has been renamed as AddReplace and in that release you can replace existing accelerators without first removing them).  You can remove a type accelerator by invoking $typeAcceleratorsType::Remove and passing in the accelerator string you want to remove.

This is a great start, because now we can customize our type accelerators, but I doubt anyone that is looking for using keyword functionality in PowerShell would feel that this was a good solution because it’s missing all of the glue that would allow you to get fast access to all types within a namespace.

### Enumerating Namespaces

Now that we’re armed with the knowledge required to manage type accelerators, let’s figure out how we can enumerate types inside namespaces so that we can get the most out of this capability.  The first step in this enumeration process is the retrieval of the assembly containing the namespace we are interested in.

The System.Reflection.Assembly type has three very useful static methods that allow you to get assembly objects: Load, LoadWithPartialName, and LoadFrom.  If you have the full name of the assembly whose namespace you want to enumerate, you can invoke the Load static method and pass in the full assembly name.  If you only have a partial assembly name (i.e. none of the Public Key token stuff that is included in a full assembly name), you can invoke the LoadWithPartialName static method and pass in the partial name (note: don’t forget to include the System qualifier for namespaces that are part of the System namespace; otherwise they simply will not load).  If you are working with an assembly that is not loaded in the Global Assembly Cache (GAC), you can use the LoadFrom static method and pass in the full path to the file containing the assembly you want to load.

For this example, I’m going to use the System.Windows.Forms namespace that I referred to earlier in this article.  This namespace contains a lot of useful functionality that enables PowerShell scripters to include some user interface capabilities in their scripts, and it’s used very often in PowerShell scripts today.  Following the description of the System.Reflection.Assembly LoadWithPartialName static method, you can load the assembly containing the System.Windows.Forms namespace by invoking this command:

```powershell
$assembly = [Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
```

With the assembly loaded, we need to enumerate the public types that belong to the System.Windows.Forms namespace.  Fortunately, this can be done with a single pipeline, as follows:

<pre>$assembly.GetExportedTypes() | `
Where-Object { `
($_.IsPublic -or $_.IsNestedPublic) -and `
($_.FullName -like ‘System.Windows.Forms.*’) `
} | Select-Object -ExpandProperty FullName</pre>

This technique works with any assembly, whether it is already loaded into PowerShell or not.  Now that we have the tools to create type accelerators combined with the tools to enumerate namespaces, we’re able to put the two together and create something close to a using statement.  I say “close” here because the using keyword is reserved, so we cannot actually create a using command in Windows Powershell 2.0, so we’ll have to come up with another appropriate command name instead.

### Defining the Type Accelerator Module

With the type accelerator management commands and namespace type enumeration commands figured out, we’re all set to package the functionality up in a script module so that anyone can leverage this capability with simple PowerShell one-liners.  The TypeAccelerator.zip file contains a TypeAccelerator script module that includes the following public commands:

Get-TypeAccelerator (alias gtx)

This command allows you to enumerate type accelerators.  You can optionally specify the Name(s) of the accelerator(s) you want to enumerate, the Type(s) for which you want to enumerate accelerators, and the Namespace(s) from which you want to enumerate accelerators.

Add-TypeAccelerator (alias atx)

This command allows you to create new type accelerators.  It requires the Name of the accelerator and the Type it should be associated with.  By default it will overwrite existing accelerators with the same name, however you can specify NoClobber to prevent any overwrites from happening.

Set-TypeAccelerator (alias stx)

This command allows you to change an existing type accelerator, or add it if it does not exist.  It requires the Name of the accelerator and the Type it should be associated with.

Remove-TypeAccelerator (alias rtx)

This command allows you to remove type accelerators.  You can either remove type accelerators by Name, by Type, or by Namespace.

Use-Namespace (alias use)

This command allows you to emulate the C# using command in PowerShell.  You can specify the namespace(s) you want to start using by Namespace name or by path (LiteralPath or Path) and Namespace name.  You can make the namespace usage temporary by passing in a ScriptBlock, in which case the namespace will only be used inside of that script block.  You can associate a shortcut to a namespace by giving it an Alias.  This command will overwrite accelerators with the same name by default, however you can prevent that from happening by specifying NoClobber.

Usage of these commands in the context of our example is as easy as a single PowerShell command.  Here are some examples showing how you can use this module in your own scripts:

```powershell
# Enumerate all accelerators
Get-TypeAccelerator

# Enumerate type accelerators in the System.Management.Automation namespace
Get-TypeAccelerator -Namespace System.Management.Automation

# Add a type accelerator
Add-TypeAccelerator -Name arraylist -Type System.Collections.ArrayList

# Remove a type accelerator
Remove-TypeAccelerator -Name arraylist

# Use the Windows.Forms namespace
Use-Namespace -Namespace Windows.Forms -ScriptBlock {
   [DialogResult]$result = [MessageBox]::Show(‘Test’)
}
```

With this module in your tool belt, you should be able to avoid a lot of unnecessary typing of type names in your PowerShell scripts.  There are also some extremely useful things you can do with this module when defining custom types…I’ll talk more about that in a future article.

That’s it from me for this time.  I hope you find this [TypeAccelerator module][1] as useful as I do.  Feel free to distribute it however you wish.

[1]: /images/TypeAccelerator.zip