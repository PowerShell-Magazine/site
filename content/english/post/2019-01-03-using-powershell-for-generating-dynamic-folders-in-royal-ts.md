---
title: Using PowerShell for generating dynamic folders in Royal TS
author: Jan Egil Ring
type: post
date: 2019-01-03T11:00:13+00:00
url: /2019/01/03/using-powershell-for-generating-dynamic-folders-in-royal-ts/
post_views_count:
  - 8378
views:
  - 8005
categories:
  - Module Spotlight
tags:
  - Scripts

---
[Royal TS][1] is a powerful tool for managing remote systems using many different protocols such as Remote Desktop (RDP), PowerShell, SSH, HTTPS, and many more. In this article we will look at a new feature introduced in Royal TS 5.0 (released in December 2018) called dynamic folders.

Previously, I have written an [article][2] covering a PowerShell module for managing Royal TS Documents which is built in to Royal TS. 

In that article I’ve showcased a script called [Update-RoyalFolder.ps1][3], which could replicate server computer objects from a specified Active Directory domain or Organizational Unit (OU).

This was very useful as the script could be scheduled to update a Royal TS connections document, for example on a daily basis.

The new Dynamic Folder Script feature allows you to configure a script and the interpreter which populates the dynamic folder content. 

We start by creating a new folder of the Dynamic Folder type:

![image](/images/royalts1.png)

Give it a meaningful name, such as the source we are going to dynamically retrieve data from:

![image](/images/royalts2.png)

On the Dynamic Folder Script window, we can choose PowerShell to be the script interpreter:

![image](/images/royalts3.png)

You will get an example script which shows what kind of objects is expected, as well as how to convert them to JSON (which is the output format Royal TS expects):

![image](/images/royalts4.png)

After modifying the options, such as credentials, click OK. When right-clicking the folder we create, we can see that we have the option of reloading the folder:

![image](/images/royalts5.png)

Clicking this will trigger the PowerShell script we just saw on the Dynamic Folder Script window, and the folder should now look like this when using the provided example script.

![image](/images/royalts6.png)

According to the [documentation][4], there are two options available for reloading a dynamic folder:

  * **_Automatically reload folder contents_**
_ &#8211; If checked, Royal TS will automatically reload the folder contents when the document is opened._
  * **_Persist (cache) folder contents_**
_ &#8211; If checked, Royal TS will save (cache) the contents of this dynamic folder within the document._

By default, none of the two options is enabled.

In order to populate the dynamic folder with our own data using PowerShell, we can build custom objects with the necessary properties:

```powershell
[PSCustomObject]@{
        Name                   = $PSItem.Name
        Type                   = 'RemoteDesktopConnection'
        ComputerName           = $PSItem.Name
        CredentialName         = 'DOMAIN\username'
        Path                   = MySubfolder
    }
```


This example creates an object to be used with a Remote Desktop Connection. If you want to build other connection types, see the documentation for [RoyalJSON and Dynamic Folders][5].

Next, we need to put all the objects we have created in a hash table called ‘Objects’, as this is what the RoyalJSON format expects:

```powershell
$RoyalTSObjects = @{}
$null = $RoyalTSObjects.Add('Objects',$Servers)
```

The final piece is to convert the hash table to JSON format, which is very convenient to do in PowerShell:

```powershell
$RoyalTSObjects | ConvertTo-Json
```


As you can see, for a PowerShell user it is very straightforward to build a script for dynamically populating a folder with connection objects in Royal TS.

We can populate it with any data we can retrieve from PowerShell.

We will start by looking at an example on how to accomplish this using Active Directory as the data source.

You need to install Remote Server Administration Tools (RSAT) in order to leverage the Active Directory module for PowerShell from a workstation. Starting with Windows 10 October 2018 Update, RSAT is included as a set of &#8220;Features on Demand&#8221; in Windows 10 itself. From PowerShell, it can be installed using this command:

```powershell
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```


A Royal TS community member has created a [YouTube video][6] explaining more details about retrieving data from Active Directory for building a dynamic folder in Royal TS, such as building the Dynamic Folder sub folder structure based on the Organizational Unit structure the computer objects is retrieved from.

You can use your existing skills for retrieving the computer accounts you want from Active Directory. I am re-using an existing function I have created called Get-ServerFromAD. This will retrieve computer accounts with an operating system name starting with Windows Server*, exclude Cluster Name Objects and include computer accounts which have logged on during the last number of days specified (30 is the default).

You can find the complete script [here][7].

I would recommend to first run the script manually in order to verify that data can be retrieved from Active Directory. You may also want to customize options such as domain name and credentials. I am using the script from a non-domain joined laptop, hence I need to specify credentials.

When the script is customized and verified, paste it in the Dynamic Folder Script window and click OK:

![image](/images/royalts7.png)

After re-loading the document, you should now see server computer accounts from Active Directory:

![image](/images/royalts8.png)

I have also created a [script][8] for getting computer names from System Center Virtual Machine Manager (both hosts and virtual machines), which can be used to populate a dynamic folder:

![image](/images/royalts9.png)

In my example script I’ve created a flat structure, putting all computer names in a subfolder called VMM. Here it is possible to do all sorts of creative things, such as use Group-Object on the OperatingSystem property and then create SSH connections for Linux machines and RDP connections for Windows machines.

I plan to create other scripts to retrieve computer names from other sources, such as Azure, Amazon Web Services, and VMware. Royal Applications has a dedicated [repository][9] for various automation scripts &#8211; created both by the Royal TS team and the community – where I also will submit Pull Requests for my contributions.

**Using PowerShell Core as Script Interpreter**

By navigating to the Royal TS Options, it is possible to modify Script Interpreter settings in the Advanced section:

![image](/images/royalts10.png)

By default, the PowerShell Script Interpreter is configured with the following path:

_%windir%\System32\WindowsPowerShell\v1.0\powershell.exe

If you rather want to leverage PowerShell Core as the engine for the PowerShell Script Interpreter, simply change the path to either pwsh.exe (as it is available in the System Path):

![image](/images/royalts11.png)

Alternatively, specify the full path:

_C:\Program Files\PowerShell\6\pwsh.exe_

I have been using PowerShell Core without issues for the 2 Dynamic Folder Scripts I have showed in this article. For the Active Directory Dynamic Folder Script, PowerShell Core is using ~2.5 seconds to run against my lab Active Directory instance, while Windows PowerShell is using ~4 seconds.

**Summary**

In this article we have looked at how the dynamic folder script feature in Royal TS can be used to dynamically creating Royal TS connection objects based on data gathered from a PowerShell script.

We also looked at different sources we can retrieve data from, such as Active Directory and System Center Virtual Machine Manager.

_Bonus tip_

If you are a Microsoft MVP, you can get an NFR license for Royal TS by sending an e-mail to support (at) royalapplications (dot) com with a link to your MVP profile.

[1]: https://www.royalapplications.com/
[2]: https://www.powershellmagazine.com/2015/01/08/introducing-the-royal-ts-powershell-module/
[3]: https://github.com/janegilring/PSCommunity/blob/master/Royal%20TS/Update-RoyalFolder.ps1
[4]: https://content.royalapplications.com/Help/RoyalTS/V5/index.html?reference_dynamicfolder_advanced.htm
[5]: https://support.royalapplications.com/support/solutions/articles/17000070210
[6]: https://www.youtube.com/watch?v=pKurlGhMfoQ
[7]: https://github.com/janegilring/PSCommunity/blob/master/Royal%20TS/Dynamic%20Folder%20Scripts/Get-ServerFromAD.ps1
[8]: https://github.com/janegilring/PSCommunity/blob/master/Royal%20TS/Dynamic%20Folder%20Scripts/Get-ServerFromSCVMM.ps1
[9]: https://github.com/royalapplications/toolbox