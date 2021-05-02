---
title: 'Writing a PowerShell module in C#, Part 1: The basics'
author: Carlos Perez
type: post
date: 2014-03-18T16:00:28+00:00
url: /2014/03/18/writing-a-powershell-module-in-c-part-1-the-basics/
categories:
  - How To
tags:
  - Modules
  - PSCmdlet
  - How To

---
In this series we will cover the basics of building a Windows PowerShell binary module using C#. In the first part of the series we will build a module with just one cmdlet called Get-Salutation that will resemble the traditional “Hello World” example.

We will use Visual Studio 2013 since it includes the reference assemblies for System.Management.Automation -- the root namespace for Windows PowerShell&#8211;for Microsoft .Net Framework 4.0. Microsoft has not released a PowerShell 4.0 SDK with reference assemblies using Microsoft .NET Framework 4.5 at this moment. If you are planning to support Windows PowerShell  2.0 then you will need to download the Windows PowerShell 2.0 SDK from Microsoft <http://www.microsoft.com/en-us/download/details.aspx?id=2560><span style="text-decoration: underline;">.</span> Visual Studio 2013 Express for Windows Desktop version can also be used.

Open Visual Studio and create a new project from the **File** menu:

![](/images/image001.gif)

Select the **Visual C#** from the installed **Templates** list and then pick **Class Library** since compiled modules in PowerShell are in DLL format:

![](/images/image002.gif)

We give it a name and specify a location for our project. For the purpose of this tutorial I will name my project MyModule.

Next step is to set the project’s minimum target .NET Framework depending on the lowest version of PowerShell we want to support. You can use as a reference:

  * PowerShell 2.0 – .NET Framework 3.5
  * PowerShell 3.0 &#8211; .NET Framework 4
  * PowerShell 4.0 &#8211; .NET Framework 4.5

![](/images/image0032.jpg)

Now, on the Application tab we select the target framework from the dropdown list.

![](/images/image0043.jpg)

For this example project we use a PowerShell 3.0 assembly and we want to support that as the lowest version of PowerShell. That’s why we select .NET Framework 4. We will be asked to confirm the change and it will re-open the project.

We need to load the **System.Management.Automation** library as a reference to our project to make the PowerShell API calls available. To do this we right click on Reference in the **Solution Explorer** and select **Add Reference**

![](/images/image003.gif)

In the **Reference Module Manager** click on **Browse**:

![](/images/image0042.jpg)

Now navigate to **C:\Program Files (x86)\Reference Assemblies\Microsoft\WindowsPowerShell\3.0** This folder is created with the installation of Visual Studio 2013. There we select the **System.Management.Automation.dll** to add it to our references.

![](/images/image005.gif)

Once we have the reference assembly in our project we can now use the assembly to get access to the API calls we need to build a PowerShell module. We start by adding **System.Management.Automation** to the default list that Class template has provided.

![](/images/image006.gif)

Those familiar with PowerShell advanced functions will notice that process for creating a cmdlet in C#  is very similar. The major difference is that a module with advanced functions is stored in a .psm1 file and cmdlets are stored in a DLL file. The attributes are used even in the same manner so for a person that has written script modules in PowerShell going to C# is very easy.

Let’s start by applying the attribute to the default class created in our namespace. You will see that in the case of C# we have to define the verb and the noun for the cmdlet. Thankfully, Visual Studio supports the IntelliSense for the verb group and the verbs inside the group.

![](/images/image007.gif)

![](/images/image008.gif)

After selecting a verb, we need to add the second parameter to the attribute&#8211;the noun we want to use (in our case a noun is Salutation). The verb and noun combination we use in the attribute is how the cmdlet will be named in PowerShell. The class name has no impact. As you can see it the following screenshot, if we place a comma after the second parameter, IntelliSense will suggest other named parameters we can use when defining the cmdlet.

![](/images/image009.gif)

One thing we have to make sure of is that the class inherits from the PSCmdlet class. This is done by adding **: PSCmdlet** at the end of the class name.

![](/images/image010.gif)

I like naming my classes **_<verb><noun>_** in Pascal case following Microsoft naming guidelines <http://msdn.microsoft.com/en-us/library/4xhs4564(v=vs.71).aspx> to easily identify them when working inside of Microsoft Visual Studio.

In PowerShell, the parameters of an advanced function become the named parameters of the command, in the case of a class that defines a cmdlet it is a public class properties that become the named parameters of the command. In the case of our Get-Salutation cmdlet we would like to be able to provide a single name or array of names we would like to get a salutation for. Also, we would like our Name parameter to have the following characteristics:

  * The parameter has to be mandatory.
  * To accept pipeline input by property name.
  * Use aliases for common property names that can be used to reference a person’s name in an object.
  * Provide a Help Message to describe it when looking at help or being asked to provide a value.

The syntax is almost identical to PowerShell. We apply the attributes to the public property and its name becomes the name of the parameter. For more information on using properties check <http://msdn.microsoft.com/en-us/library/w86s7x04.aspx>

![](/images/image011.gif)

A PowerShell function can have the following named script blocks:

  * **Begin** &#8211; This block is used to provide optional one-time pre-processing. In the case of multiple values provided through the pipeline this block only executes once.
  * **Process** &#8211; This block executes each time the command is called. In the case of multiple values provided through the pipeline this block executes for each one of those values.
  * **End** &#8211; This block is used to provide optional one-time post-processing.

When we write a cmdlet in C# we accomplish the same using the following methods:

  * **BeginProcessing()** &#8211; Provides a one-time, preprocessing functionality for the cmdlet.
  * **ProcessRecord()** &#8211; Provides a record-by-record processing functionality for the cmdlet.
  * **EndProcessing()** &#8211; Provides a one-time, post-processing functionality for the cmdlet.

Each of the methods is inherited from the PSCmdlet class so we need to use the **protected** and **override** modifiers.

![](/images/image012.gif)

The list of all supported methods is available at <http://msdn.microsoft.com/en-us/library/system.management.automation.cmdlet_methods(v=vs.85).aspx> . Since we are building a super simple module just to show the basic concepts we only need the **ProcessRecord()** method (we won’t need to initialize any data or finalize any action or actions at the end of execution). Also, we’ll use the **WriteVerbose()** method to write verbose information if requested. In an advanced function this would be the equivalent of using the **_Write-Verbose_** __cmdlet. Next, a string object is created and returned to pipeline using the **WriteObject()** method. This is equivalent to placing a variable containing an object or collection of objects on its own in the body of an advanced function so it would be send down the pipeline.

![](/images/image013.gif)

Now we just need to build the solution by pressing **F7** or selecting **Build Solution** from the **BUILD** menu.  In the Output pane of Visual Studio we should see that it builds successfully and provides us a path to the generated DLL.

![](/images/image014.gif)

To test the module we open a PowerShell session, navigate to where the DLL is and use the **Import-Module** cmdlet with the -Verbose parameter to see if it loads our cmdlet.

![](/images/image015.gif)

If we use the **Get-Help** cmdlet against Get-Salutation we should see the Name parameter and the proper information for it.

![](/images/image016.gif)

Let’s test the cmdlet directly and from the pipeline to make sure it works like it should. First, we test giving it several values to the parameter, then a collection of strings from the pipeline and at the end, an object with property that matches one of the aliases.

![](/images/image017.gif)

In the next part we will look at setting up debugging for the module inside of Visual Studio 2013.

If you are interested in more advanced examples for PowerShell 3.0 and 4.0 take a look at the sample pack in MSDN <http://code.msdn.microsoft.com/Windows-PowerShell-30-SDK-9a34641d>