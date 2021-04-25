---
title: Automating Configuration Manager with Windows PowerShell
author: Trevor Sullivan
type: post
date: 2012-04-16T18:00:28+00:00
url: /2012/04/16/automating-configuration-manager-with-windows-powershell/
aktt_notify_twitter:
  - yes
views:
  - 26470
post_views_count:
  - 2402
categories:
  - Configuration Manager
tags:
  - Configuration Manager

---
### Introduction

First off, I'd like to express my sincere thanks to the board of the PowerShell Magazine for allowing me to publish an article in the premiere issue of the magazine. It’s quite an honor to be asked to participate in such an endeavor, and I can only wish to live up to the high expectations that other PowerShellers have set through their demonstrable experience.

In this article, we’ll explore the use of Microsoft’s Windows PowerShell (aka. PoSH) in concert with Microsoft System Center Configuration Manager (SCCM or ConfigMgr). ConfigMgr is Microsoft’s flagship tool for managing systems large-scale PowerShell is the ideal automation tool for repeatable tasks around ConfigMgr. Although SCCM is a powerful management tool in and of itself, there are often situations that require custom scripting to accomplish a task that would otherwise take a lengthy amount of time if performed manually.

### Why use PowerShell with ConfigMgr?

At this point, you might be asking yourself: “why is PowerShell so important if I’m automating ConfigMgr? I use VBscript or don’t need to script at all.”

First and foremost, it’s because PowerShell is Microsoft’s current, standard automation tool for their entire platform. Nearly every single Microsoft product is easily automated with PowerShell, due to the vast array (no pun intended) of extensibility options that PowerShell has. It plugs directly into .NET types (and even allows dynamic compilation of C# code), Windows Management Instrumentation (WMI), Component Object Model (COM), Web Services (SOAP, REST), and so on. Along with this reasoning, is the fact that VBscript is outdated and hasn’t seen any significant update by Microsoft in years — if you’re still using VBscript, I’d recommend taking the time it takes you to code, and invest it in learning PowerShell. In the long run, it’ll save you a lot of time, and turn you into a much more powerful and capable automator!

Thanks to the out-of-box commands in PowerShell, some automation tasks requiring many lines of VBscript code can be consolidated into much fewer lines of PowerShell code. To be honest, sometimes I feel a little dirty using cmdlets, as I personally prefer using PowerShell as a “.NET scripting language” (meaning I like to write C#-like code). As long as they’re built-in commands though, and don’t create a dependency on an external module, I’m usually ok with using them.

Most importantly, using PowerShell with ConfigMgr is important because the ConfigMgr “API” is essentially a huge WMI provider. As you probably know by now, PowerShell is tightly integrated with WMI and provides a number of type accelerators and cmdlets to make retrieving, changing, and monitoring information in WMI easy.

You might even say that you don’t need to script ConfigMgr (or anything) at all, which may be true, however I’d argue differently. In my personal experience, I have found that knowledge of automation has provided me many more opportunities to tell people “yes” rather than “no” – this results in you becoming a more valuable asset to your employer and your peers in the IT community at large. Have you noticed how a lot of people take other people’s scripts and tweak them to their needs? Wouldn’t it be cool if \*you\* were one of those people whose scripts are taken and tweaked?

Now that we’ve discussed the “whys” of using PowerShell with ConfigMgr, let’s move on and talk about how to get started.

### How do I get started with automating ConfigMgr and PowerShell?

#### WMI

To get started, the most important concept you’ll need to familiarize yourself with is WMI. With a firm understanding of WMI, your experience scripting ConfigMgr will be much simpler. There are a number of books, and plenty of blogs and articles out there that either talk explicitly about WMI or at least show examples of how to use WMI with PowerShell.

As far as PowerShell goes, you’ll be using the Get-WmiObject cmdlet a lot, as well as the [wmi] type accelerator. In some cases, you’ll also be able to leverage the [wmiclass] type accelerator when you want to retrieve metadata about WMI class definitions or properties.

##### WMI Explorer

You should also equip yourself with the proper developer tools when writing ConfigMgr PowerShell scripts.

Aside from PowerShell itself, one of the most vital tools I always keep handy is a WMI explorer utility. Windows has a built-in WMI explorer called wbemtest, which is both powerful, flexible, and comprehensive in WMI feature support. Wbemtest is great when you are working on a computer that doesn’t have anything else immediately available, and for testing event queries, but for general browsing through WMI, I prefer to use the free WMI Explorer utility from SAPIEN Technologies. Although it does not provide any support for testing WMI event queries, it’s great to be able to easily list WMI classes, and view their properties and methods (complete with input and output parameters).

![](/images/wbem.png)

#### Script Editor

For testing PowerShell code, you can obviously use the console, but when writing longer scripts or modules, you’ll want an editor of sorts, similar to how developers use Visual Studio for .NET projects. For this, you have the option of using the built-in PowerShell v2 Integrated Scripting Editor (ISE) or you can use the tool that I prefer, the free Quest PowerGUI Script Editor available at [http://www.powergui.org][1]. Both the ISE and PowerGUI are extensible editors, however I tend to prefer PowerGUI for the “intellisense” support similar to what Visual Studio offers. It’s pretty much up to you to make a decision on which editor feels best to you.

### What do I need to be aware of?

Although ConfigMgr’s API is built on WMI, there are a couple of unique features that you ought to be aware of. Even if you’ve worked with PowerShell and WMI before, some things in the ConfigMgr provider are unique to it, and will require special consideration.

### Correlation to SQL Views

As you work with the ConfigMgr WMI provider, you’ll begin to notice a strong correlation between the ConfigMgr database’s more friendly SQL views and the WMI classes offered by the provider. In most cases, the only major difference is going to be the item’s prefix – for example, the hardware inventory SQL views are prefixed with “v\_GS\_” whereas the correlating WMI classes are prefixed with “SMS\_G\_System_”.

As a result of this strong — but not entirely consistent — correlation, it is generally a simple task to translate SQL queries into WQL queries.

### Lazy Properties

Certain ConfigMgr WMI classes will have properties marked with a WMI qualifier called “lazy.” If a property is marked lazy, then it means that the property’s value will not be returned in an enumeration of WMI class instances. In this case, to retrieve the property’s value, you must first determine the specific WMI instance you want, and obtain a reference to that instance directly using its WMI path. I’ve found that the easiest method of achieving this is to enumerate the instances of a given WMI class until a condition I am interested in is met, and then use the [wmi] type accelerator to access the WMI instance using its __PATH system property.

```powershell
$SccmUpdateLists = Get-WmiObject -Namespace root\sms\site_lab `
            -Class SMS_AuthorizationList –ComputerName sccm01

foreach ($SccmUpdateList in $SccmUpdateLists)
{
   if ($SccmUpdateList.LocalizedDisplayName -eq ‘My Update List’)
   {
      $SccmUpdateList = [wmi]”$($SccmUpdateList.__PATH)”
   }
}
```

On my personal blog at [http://trevorsullivan.net][2], I have an article that talks about lazy properties in depth and also describes how to identify which classes have lazy properties, and exactly which properties they are.

### ConfigMgr-specific WQL Keywords

Another unique attribute of the ConfigMgr WMI provider is the support for additional WQL keywords. The most notable and probably most commonly used keyword that you’ll get accustomed to using is the JOIN keyword. Since the ConfigMgr provider does not use CIM_REFERENCE types with association classes to join other data classes together, it’s up to you to use a common identifier to join data classes together. For ConfigMgr hardware inventory classes, you’ll generally join in the ResourceID property, and other objects use different common identifiers, such as UpdateID for software updates objects.

Here is an example of using the JOIN keyword:

```powershell
PS> $MyQuery = @""

>> select * from SMS_G_System_Computer_System
>> JOIN SMS_G_System_Disk on
>> SMS_G_System_Computer_System.ResourceID = SMS_G_System_Disk.ResourceID
>> @

PS> Get-WmiObject -Namespace root\sms\site_lab -Query $MyQuery
```

Rather than typing the query by hand, you can optionally use the query builder in the ConfigMgr collection editor window, to assist you with visually building certain inventory-related queries. Once the desired query is built, you can simply hit the “Show query language” button to display the WQL code, which you can then copy/paste into your PowerShell script.

### Event Management

One of the more powerful (and lesser known) functions of WMI is the ability for you to subscribe to and respond to events. Since the ConfigMgr API is built on WMI, we automatically get this functionality for free. Although ConfigMgr generates certain status messages to notify administrators of changes to objects in the hierarchy, it is not comprehensive. In situations where these status messages do not get generated, it may be necessary to subscribe for WMI events in order to get the desired change notifications.

Using PowerShell v2, it is possible to create a temporary event registration that survives as long as the PowerShell session is open, or until explicitly unregistered, using the Register-WmiEvent cmdlet. If a more permanent notification solution is needed, you can create permanent WMI event registrations using the [PowerEvents][3] PowerShell module. Back in November 2010, I created this module partially with the intention of using it to ease registration of permanent change notifications in ConfigMgr.

If you want to learn more about WMI event management, please review the documentation included in the aforementioned PowerEvents module.

### Challenges

As with anything, there are certainly pitfalls to be aware of, especially as a developer. We’ll review a couple of these below.

#### Site Control File

In Configuration Manager 2007 and prior versions of the product, there was a concept known as the Site Control File (SCF). Each primary site has a SCF and stores most of the site-level configuration data, such as site boundaries, sender addresses, client agent configuration settings, site server roles, and so on. The SCF is quite challenging to work with in PowerShell, and as such, I would recommend sticking with VBscript or C# examples in the Configuration Manager SDK.

#### Lazy Properties

We discussed lazy properties before, and how to retrieve them. Something you should concern yourself with, when working with classes that have lazy properties, is that changing and committing a WMI instance without first retrieving its lazy properties will effectively nullify the values of the lazy properties.

#### WMI References

In other WMI providers, associations between objects are often made using the CIM_REFERENCE type, which was mentioned in the earlier section on ConfigMgr-specific WMI features. This means that you won’t be able to use the “REFERENCES OF” or “ASSOCIATORS OF” WQL queries to discover objects that are related to a WMI class or specific instance in the ConfigMgr provider.

### Conclusion

In this article, we have reviewed the benefits of using Microsoft Windows PowerShell to automate the Microsoft System Center Configuration Manager product, and some of the things to watch out for. Through automation of ConfigMgr, you can achieve stronger, consistent management of your infrastructure, save yourself lots of time, perform governance audits, and run jobs on a schedule if necessary. I wish you the best in your scripting of ConfigMgr, and hope you can apply the concepts from this article practically going forward.

Finally, feel free to reach out to me at <pcgeek86@gmail.com> if you have any questions regarding PowerShell, WMI, or System Center Configuration Manager. I would also recommend the MyITforum discussion forums and mailing lists to assist you with any ConfigMgr questions, as there is a very strong community there around this product.

[1]: http://www.powergui.org/
[2]: http://trevorsullivan.net/
[3]: http://powerevents.codeplex.com/