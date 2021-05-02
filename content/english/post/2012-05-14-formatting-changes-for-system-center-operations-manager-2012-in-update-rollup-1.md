---
title: Formatting changes for System Center Operations Manager 2012 in Update Rollup 1
author: Stefan Stranger
type: post
date: 2012-05-14T18:00:18+00:00
url: /2012/05/14/formatting-changes-for-system-center-operations-manager-2012-in-update-rollup-1/
categories:
  - How To
  - System Center
tags:
  - System Center
  - How To

---
On the 7th of May 2012 we released our first Update Rollup 1 (UR1) for System Center 2012. In addition to earlier released fixes for Virtual Machine Manager (VMM) and App Controller (AC), the update rollup now includes fixes for System Center Operations Manager (OM).

And because I&#8217;m a [Premier Field Engineer at Microsoft][1] specialized in System Center Operations Manager I was curious to see if some of the suggestions I made in pre-RTM and Beta versions where part of this Update Rollup 1. Let&#8217;s go back some months when I was in Redmond for our semi-annual TechReady. TechReady is  5-day internal technical conference for Microsoft employees. During my time in Redmond for TechReady I was asked to work with [James Brundage][2] on some of the suggestions I made for the System Center Operations Manager cmdlets in the pre-RTM and Beta versions of the product. [System Center Operations Manager 2012][3] is officially released during the Microsoft Management Summit in Las Vegas in April 2012.

When looking at the [Description of Update Rollup 1 for System Center 2012][4] I found the following:

![](/images/scom2012_01.png)

Yes! There I found the formatting changes James and I worked on during my week at TechReady. But let&#8217;s explain first some basic information about PowerShell formatting. We start using our beloved Get-Help cmdlet.

Get-Help about\*format\* will return Help information from the about_Format.ps1xml help file. It tells us that the Format.ps1xml files in Windows PowerShell define the default display of objects in the Windows PowerShell console. You can create your own Format.ps1xml files to change the display of objects or to define default displays for new object types that you create in Windows PowerShell.

And that&#8217;s exactly what James did with the suggestions I made for some of the earlier formatting in the Beta releases of System Center Operations Manager 2012, changing the format.ps1xml files for System Center Operations Manager 2012. How do we know what has changed since the last RTM Release? Just compare the Release to Manufacturing (RTM) formatting files with the new UR1 formatting files.

To find the format.ps1xml files for the Operations Manager 2012 we need to look in the installation where the Operations Manager Console is installed. We can search the program installation folder recursively for the files with the format.ps1xml extension.

<pre>PS&gt; $path = "D:\Program Files\System Center 2012\Operations `
          Manager\Powershell\OperationsManager"
PS&gt; Get-ChildItem -Path $path -Recurse -Filter *.format.ps1xml</pre>

* I&#8217;ve installed Operations Manager on my D: drive, default it&#8217;s installed on your C: drive.

![](/images/scom2012_02.png)

Now we know which files we need to compare with the new installed *.format.ps1xml from the UR1.  We can use the Compare-Object cmdlet to check which files have been changed.

```
$PreUR1FilesPath = "D:\Temp\Powershell_preRU1"
$PostUR1FilesPath = "D:\Temp\Powershell_postRU1"
$filter = "*.format.ps1xml"

$PreUR1FormatFiles = Get-ChildItem -Path $PreUR1FilesPath -Recurse -Filter $filter
$PostUR1FormatFiles = Get-ChildItem -Path $PostUR1FilesPath -Recurse -Filter $filter
$result=Compare-Object -ReferenceObject $PreUR1FormatFiles -DifferenceObject `
          $PostUR1FormatFiles -Property Name, Length
$result | Format-Table -AutoSize
```

![](/images/scom2012_03.png)

The only file that seems to be changed is the Microsoft.SystemCenter.OperationsManagerV10.format.ps1xml file.

![](/images/scom2012_04.png)

And again we are using the Compare-Object cmdlet to compare the content of the pre RU1 Microsoft.SystemCenter.OperationsManagerV10.format.ps1xml file with the post RU1 format file.

<pre>$strReference = Get-Content "D:\Temp\Powershell_preRU1\OperationsManager\ `
   OM10.Commands\Microsoft.SystemCenter.OperationsManagerV10.format.ps1xml"
$strDifference = Get-Content "D:\Temp\Powershell_postRU1\OperationsManager\ `
   OM10.Commands\Microsoft.SystemCenter.OperationsManagerV10.format.ps1xml"
Compare-Object -ReferenceObject $strReference -DifferenceObject $strDifference</pre>

![](/images/scom2012_05.png)

We can see all the formatting that has been changed in RU1.  To be honest there are better tools to compare the contents of two files, and I&#8217;ve use Notepad++ to look at one interesting formatting change 😉

![](/images/scom2012_06.png)

We can see formatting for the Microsoft.EnterpriseManagement.Administration.NotificationRecipient TypeName did not existed before RU1 was installed. To find the cmdlet(s) which has the Microsoft.EnterpriseManagement.Administration.NotificationRecipient is easy using the Get-Command cmdlet.

<pre>Import-Module OperationsManager
Get-Module
Get-Command -Module OperationsManager |
 Where-OBject {$_.OutputType -like `
  "Microsoft.EnterpriseManagement.Administration.NotificationRecipient"}</pre>

![](/images/scom2012_07.png)

We can now run the Get-SCOMNotificationSubscriber cmdlet from the Operations Manager Shell to see the cool formatting improvements in RU1.

![](/images/scom2012_08.png)

This is it for now about the formatting improvements in RU1 for System Center Operations Manager 2012. Maybe next time a blog post about how you can use the ** **[**EzOut module**][5] from Start-Automating for simplifying the process of making format XML files. You can use EZOut to change formatting and type information on the fly, or use it to help author files to use with your PowerShell modules.

References:

  * [Update Rollup 1 for System Center 2012 &#8211; Operations Manager][6]
  * Detailed list of fixes can be found on Knowledge Base Article [KB2686249][4]
  * System Center Operations Manager Team blog [Update to Update Rollup 1 for System Center 2012][7]

[1]: http://blogs.technet.com/b/stefan_stranger/
[2]: http://twitter.com/jamesbru
[3]: http://www.microsoft.com/en-us/server-cloud/system-center/default.aspx
[4]: http://support.microsoft.com/kb/2686249
[5]: http://ezout.start-automating.com/
[6]: http://www.microsoft.com/en-us/download/details.aspx?id=29697
[7]: http://blogs.technet.com/b/momteam/archive/2012/05/09/update-to-update-rollup-1-for-system-center-2012.aspx