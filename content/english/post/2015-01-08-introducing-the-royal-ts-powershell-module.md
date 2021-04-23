---
title: Introducing the Royal TS PowerShell module
author: Jan Egil Ring
type: post
date: 2015-01-08T20:02:36+00:00
url: /2015/01/08/introducing-the-royal-ts-powershell-module/
views:
  - 24109
post_views_count:
  - 3923
categories:
  - Module Spotlight
  - Royal TS
tags:
  - Modules
  - Royal TS

---
Royal TS is one of many applications on the market for managing Remote Desktop connections. There are also many other features in Royal TS and you can read more about them on their [website][1].

In this article, we are going to focus on the new PowerShell module for managing Royal TS Documents.

The PowerShell module, introduced in the beta version of Royal TS 3.0, is available [here][2].

### Getting started

If you install the MSI file available on the website for the beta version, you can import the module as follows:

![](/images/royal1.png)

If you download the ZIP file, you will need to specify the path to the location where the extracted RoyalDocument.PowerShell.dll resides. In the final version of Royal TS 3.0, the module will hopefully be installed to a proper path specified in $env:PSModulePath variable, so that we don&#8217;t need to specify the path manually.

After the module is successfully imported, we can explore the available cmdlets:

![](/images/royal2.png)

There are 17 cmdlets in the initial version of this module.

There aren&#8217;t any help available yet, so we can’t use Get-Help to get more information about the cmdlets. However, there are some information in the RoyalTS3.chm file in the Royal TS program folder to help us getting started:

![](/images/royal3.png)

Next step is to create a Royal Document where we can store connections. As described in the Getting Started section of the help file, we first need to create a new RoyalStore in memory before we can use New-RoyalDocument.

In order to store the in-memory document to disk we need to use the Out-RoyalDocument cmdlet:

![](/images/royal4.png)

### Creating Remote Desktop connections

Before we go further, you might want to see how the UI looks like. Here is a screenshot of Royal TS 3.0 beta connected to a Server Core instance using Remote Desktop:

![](/images/royal5.png)

What we are going to create next is the Servers folder and Remote Desktop connections on the left side.

![](/images/royal6.png)

The inline comments should describe what is happening. When the code is executed, Royal TS opens up, and we can see the new folder and Remote Desktop connection is present:

![](/images/royal7.png)

If we right click the Remote Desktop connection we created and go into Properties, we can also see that the Computer Name is configured with the value we supplied as the URI:

![](/images/royal8.png)

In the Properties dialog, we can see that there are many options available to configure, such as Remote Desktop Gateway, Window Mode and so on. All of these properties are available on the object we created; a subset of them is visible in the screenshot below:

![](/images/royal9.png)

An object of the type RoyalRDSConnection is returned when we call the New-RoyalObject and the Set-RoyalObjectValue cmdlets.

Next, let&#8217;s see how we can configure a couple of these properties as an example:

![](/images/royal10.png)

![](/images/royal11.png)

We&#8217;ll configure the Credential Configuration to “Use credentials from the parent folder” and the Resize mode to “Smart Sizing”:

![](/images/royal12.png)

When the code has been executed, the changes should be reflected in the UI:

![](/images/royal13.png)

![](/images/royal14.png)

The code shown in the examples in this article is available in [this Gist][3].

### Practical example

The ability to automate Royal TS documents using PowerShell opens up many interesting scenarios.

I have created a script which demonstrates a practical example. The following is copied from the script&#8217;s comment-based help:

_Syntax_

```powershell
.\Update-RoyalFolder.ps1 [-RootOUPath] <String> [[-RoyalDocumentPath] <String>] [-RemoveInactiveComputerObjects ] [[-InactiveComputerObjectThresholdInDays] <String>] [-UpdateRoyalComputerProperties ] [-UpdateRoyalFolderProperties ]  [[-RTSPSModulePath] <String>] [<CommonParameters>]
```

_Description_

_    This script will mirror the OU structure from the specified root OU to a top level folder in the specified Royal TS document and create Remote Desktop Connection objects for all AD computer accounts meeting specific criteria._

_       The criteria are the following:_

_       -The computer object is registered with a Server operating system (the object&#8217;s &#8220;Operatingsystem&#8221; LDAP property meets the filter &#8220;Windows Server*&#8221;)_

_       -The computer object is not a Cluster Name Object (the object&#8217;s &#8220;ServicePrincipalName&#8221; LDAP property does not contain the word MSClusterVirtualServer)_

_       -The computer account has logged on to the domain in the past X number of days (X is 60 days if the parameter InactiveComputerObjectThresholdInDays is not specified)_

_       The purpose of this script is to show how the Royal TS PowerShell module available in Royal TS 3.0 beta can be used to manage a Royal TS document. Thus it must be customized to meet specific needs, the script shows how to configure a couple of Remote Desktop connection properties as an example._

_       The script is meant to be scheduled, for example by using PowerShell jobs or scheduled tasks, in order to have an updated Royal TS document based on active computer accounts in one or more specified Active Directory OU(s)._

_       For smaller environments, it may be appropriate to specify the domain DN as the root OU, but this is not recommended for larger environments. Instead the script may be run multiple times with different OUs specified as the root OU._

_Examples_

_    &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211; EXAMPLE 1 &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;_

_    C:\PS>& C:\MyScripts\Update-RoyalFolder.ps1 -RootOUPath &#8216;OU=Servers,DC=lab,DC=local&#8217; -RoyalDocumentPath C:\temp\Servers.rtsz_

_    Mirrors the OU structure in the C:\temp\Servers.rtsz Royal TS document based on computer accounts from the root OU OU=Servers,DC=lab,DC=local_

_    &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211; EXAMPLE 2 &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;_

_    C:\PS>& C:\MyScripts\Update-RoyalFolder.ps1 -RootOUPath &#8216;OU=Servers,DC=lab,DC=local&#8217; -RoyalDocumentPath C:\temp\Servers.rtsz -RemoveInactiveComputerObjects –UpdateRoyalComputerProperties -UpdateRoyalFolderProperties -InactiveComputerObjectThresholdInDays 30_

_    Mirrors the OU structure in the C:\temp\Servers.rtsz Royal TS document based on computer accounts from the root OU OU=Servers,DC=lab,DC=local_

_   Removes computer accounts already present in the Royal TS document folder, which have not logged on to the domain for the last 10 days._

_   Enables Smart sizing and inheritance of credentials from the parent folder for existing objects if not already enabled._

This is a screenshot of an OU structure from a test environment where I ran the demo script:

![](/images/royal15.png)

Here is the same OU structure mirrored in Royal TS, created using script:

![](/images/royal16.png)

The PowerShell script is available on [GitHub][4].

### Suggestions for improvements

If you have suggestions for improvements to the Royal TS PowerShell module, feel free to submit them in the comments section. I&#8217;ve already sent the following suggestions to the authors:

  * Make it easier to import the module, for example, by adding the path to $env:PSModulePath.
  * Add IntelliSense for parameters with predefined values, for example, for New-RoyalObject&#8217;s Type parameter.
  * Add [updatable help][5] for the Royal TS PowerShell module.
  * Add the Royal TS installation package to [OneGet][6]/Chocolatey
  * Add the Royal TS PowerShell Module to the [PowerShell Gallery][7].

### Usage scenarios

There are many possible ways to leverage the PowerShell module in order to automate the configuration of Royal TS documents, for example:

  * Create a DSC Resource for managing the configuration document, and apply a DSC configuration to an IT Pro&#8217;s desktop computer
  * Create an SMA runbook to create a central Royal TS document
  * Create a regular PowerShell script and run it on a regular basis using a PowerShell scheduled job or a scheduled task to create a central Royal TS document

Feel free to share your views on how to leverage the module in a real world scenario.

[1]: http://www.royalts.com/main/home/win.aspx
[2]: http://support.royalapplications.com/knowledgebase/articles/380700-royal-ts-v3-beta-version
[3]: https://gist.github.com/janegilring/91f174491e498e170981
[4]: https://github.com/janegilring/PSCommunity/blob/master/Royal%20TS/Update-RoyalFolder.ps1
[5]: http://msdn.microsoft.com/en-us/library/hh852754%28v=vs.85%29.aspx
[6]: https://github.com/oneget/oneget
[7]: https://www.powershellgallery.com/