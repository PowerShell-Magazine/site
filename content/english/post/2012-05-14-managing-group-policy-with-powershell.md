---
title: Managing Group Policy with PowerShell
author: Darren Mar-Elia
type: post
date: 2012-05-14T18:00:14+00:00
url: /2012/05/14/managing-group-policy-with-powershell/
views:
  - 96180
post_views_count:
  - 18101
categories:
  - Group Policy
tags:
  - Group Policy

---
### Overview

In this article, I’ll talk about your options when it comes to managing Group Policy using PowerShell. To be sure, depending upon your needs, Group Policy is nearly a full citizen in the world of PowerShell-based management. I’ll talk about why I say, “nearly” a little later, but to review, you have the following options for managing GP with PowerShell today:

  * Windows Server 2008 R2 and Windows 7 introduced the Group Policy PowerShell Module 
      * 25 cmdlets for managing Group Policy
      * You can still access the “old” GPMC APIs through COM in PowerShell
      * My free GPMC cmdlets (introduced originally in 2008) – [www.sdmsoftware.com/freeware][1] 
          * Provides a friendly wrapper on the GPMC COM APIs for Windows XP/Server 2003 environments
          * My free cmdlets for troubleshooting/reporting/managing other aspects of Group Policy with PowerShell <http://www.sdmsoftware.com/freeware> & <http://www.gpoguy.com/Free-GPOGuy-Tools.aspx>

Let’s start off by talking about the Group Policy Module that Microsoft shipped in Windows 7 & Windows Server 2008 R2.

### The Group Policy PowerShell Module

With the release of Windows 7 and Windows Server 2008 R2, Microsoft shipped the Group Policy Module—a set of 25 PowerShell cmdlets that it made available for GPO administrators to manage many of the same tasks that they would perform using GPMC. In fact, the GP PowerShell module is automatically installed when GPMC is installed as part of the Remote Server Administration Tools (RSAT) installation, as shown in Figure 1 below:

![](/images/darren1.png)

Once GPMC is installed, you can load up the Group Policy module simply by opening a PowerShell console session and typing:

<pre>PS&gt; Import-Module GroupPolicy</pre>

To get a list of cmdlets available in the module, simply type:

<pre>PS&gt; Get-Command –Module GroupPolicy</pre>

The Group Policy module covers the following tasks that you would typically perform within the GPMC GUI:

  * Creating/Deleting/Renaming/Getting GPOs
  * Backup/Restore/Copy/Import GPOs
  * Creating/Getting Starter GPOs
  * Getting/Setting GP Inheritance
  * Getting/Setting GPO Permissions
  * Creating/Deleting/Modifying GPO Links
  * Get GPO Settings and RSoP Reports
  * Getting/Setting/Deleting Administrative Template Settings
  * Getting/Setting/Deleting GP Preferences Registry Policy

Of course, there are things you <span style="text-decoration: underline;">can’t</span> ****do with the Group Policy module, such as:

  * Can’t get GPO links by SOM or GPO*
  * Can’t set WMI filters, except by GPO (and then only Gets are supported)
  * Can’t write Deny ACEs onto GPO permissions
  * Can’t write SOM delegation (e.g. Linking/RSOP/Planning permissions)*
  * Can’t read list of existing GPMC GPO Backups*

  * Can’t read/write GPOs setting outside of Administrative Templates or GP Preferences Registry Policy*

The items listed above with * are functions that you CAN do with a combination of SDM Software’s GPMC cmdlets (more on this later) or the commercial <a href="http://www.sdmsoftware.com/products/group-policy-automation-engine/" target="_blank">Group Policy Automation Engine</a>.

#### Using the GP Module

Despite the limitations in the Microsoft Group Policy, there is still quite a bit you can do with it. For example, the following PowerShell one-liner creates a new GPO, set an Administrative Template policy item, then changes delegation on the GPO to control who processes it, and finally links it to an OU:

<pre>$key = HKLM\Software\Policies\Microsoft\Internet Explorer\Restrictions'
New-GPO 'IE No Help Policy' | Set-GPRegistryValue -Key $key `
-ValueName 'NoHelpMenu' -Type DWORD -Value 1 | Set-GPPermissions -Replace `
-PermissionLevel None -TargetName 'Authenticated Users' -TargetType group | `
Set-GPPermissions -PermissionLevel gpoapply -TargetName 'Marketing Users' `
-TargetType group | New-GPLink -Target 'OU=Marketing,DC=cpandl,DC=com' –Order 1</pre>

Let’s walk through what this one-liner is doing. The first cmdlet call is to **New-GPO**, where we create a GPO called “IE No Help Policy”. The next cmdlet, called **Set-GPRegistryValue**, is the one that sets an Administrative Template policy value within my newly created GPO. You’ll notice that the parameters on this cmdlet set the underlying registry value of the Admin. Template policy in question, rather than a “friendly” path as you would see in GP editor. This is by design. In order to use this cmdlet, you’ll need to know the underlying Registry key, value and value type for a particular Admin . Template policy before you can set it using this policy. In other words, the registry key and value I used above, HKLM\Software\Policies\Microsoft\Internet Explorer\Restrictions \NoHelpMenu corresponds to a path under Administrative Templates in GP Editor – specifically Computer Configuration\Policies\Administrative Templates\Windows Components\Internet Explorer\Turn off displaying the Internet Explorer Help Menu and I had to determine that by looking at the underlying ADMX file to see which registry location set that policy.

After we set the Registry policy item, we call **Set-GPPermissions** to remove the Authenticated Users ACE from the GPO’s security delegation. This is the default ACE that gets applied to a GPO that allows all users and computers to process that GPO. In our example above, we want this GPO to only be processed by members of the “Marketing Users” group. As a result, we pipe to the next Set-GPPermissions call to add the Marketing Users Group with the Apply Group Policy (gpoapply) permission to grant that access.

Finally, we link the new GPO using **New-GPLink** to the Marketing OU within the cpandl.com domain, and we’re done! We now have a fully functional, fully permissioned, and linked GPO.

#### Getting Around Get-GPO

One of the most useful cmdlets in the Group Policy module is the **Get-GPO** cmdlet. This is the cmdlet you’ll use to get information about individual GPOs. You can also pass it the **–All** parameter to get back information on all GPOs within a given domain, as shown in **Figure 2** below:

![](/images/darren2.png)

As you can see from Figure 2, Get-GPO returns a set of properties related to the GPO in question. These range from the GPO’s name, GUID (Id property), Description (comment) and GPO status, to its version information and whether it has any WMI filters linked to it. If you get a GPO and pipe that to Get-Member, you’ll also see a wide variety of methods available to this cmdlet, as shown in **Figure 3** below:

![](/images/darren3.png)

What’s interesting about this is that many of the methods shown in Figure 3 above are wrapped up in other cmdlets with the Group Policy Module. For example, the Backup method, which could be called directly here, is exposed via the **Backup-GPO** cmdlet. But there are some methods that are not exposed through cmdlets, such as IsAclConsistent() and MakeAclConsistent(), which actually check whether the given GPO’s ACLs are consistent between the AD and SYSVOL parts of the GPO and, if not, will fix it.

In addition, we can use this Get-GPO cmdlet to get information about WMI filters that are linked to a GPO. For example, I can view a WMI filter linked to a GPO as follows:

<pre>(Get-GPO 'WMI Filter Test').WmiFilter</pre>

The output of this, as an example, is shown here:

<pre>(Get-GPO 'wmi filter test').WmiFilter | Format-List</pre>

Description : This returns a True if the timezone is Pacific

<pre>Name: Timezone
Path: MSFT_SomFilter.ID="{A1B22257-AF6E-4635-99B0-56AF0CC05E44}",Domain="cpandl.com"</pre>

You can also get details about this WMI filter, such as the query it implements, by issuing the following modification to the command above:

<pre>(Get-GPO 'wmi filter test').WmiFilter.GetQueryList()</pre>

Which returns the following:

<pre>root\CIMv2;Select * from Win32_SystemTimeZone Where DaylightName="Pacific Daylight Time"</pre>

### Returning GPO Reports

The last area I’ll focus on around the Group Policy module are some of the reporting capabilities that are provided—specifically the **Get-GPOReport** cmdlet for returning information about GPO settings and **Get-GPResultantSetOfPolicy** report for returning RSoP data against a given client.

Get-GPOReport is designed to mimic the detail you get when you click on a GPO’s settings tab within GPMC. In fact, you can output the results of this cmdlet to either HTML or XML, where the HTML mimics precisely what you see within GPMC and the XML returns a stripped down representation of settings data, without the labels that identify the settings within GP Editor. Creating a GPO settings report is as simple as typing the following PowerShell command:

<pre>Get-GPOReport 'Default Domain Policy' -ReportType html -Path c:\data\ddreport.html</pre>

In this example, I’ve created an HTML file of the Default Domain Policy GPO and stored it in a file called c:\data\ddreport.html. If I left out the –Path parameter, the raw HTML would be sent to the pipeline. Similarly, if I set the -ReportType parameter to XML, and leave off the path file, I can get the raw XML sent to the pipeline.

This provides some interesting options for getting at this XML data using PowerShell’s type accelerator. For example, I can issue the following command:

<pre>$report = Get-GPOReport 'Default Domain Policy' -ReportType xml</pre>

Once you have the report in XML format, you can navigate the XML’s nodes using standard PowerShell properties, as shown here:

![](/images/darren4.png)

You’ll find the actual settings within the report under the Computer and User properties, organized by client side extension (e.g. in Figure 4 above where the Security and Registry extensions are shown under the ExtensionData property).

Similarly, you can get RSoP data for a given computer and user by calling the Get-GPResultantSetOfPolicy cmdlet as follows:

<pre>Get-GPResultantSetOfPolicy -ReportType xml -Computer ‘win7-x86-1’ `
    -User ‘cpandl\darren’ -Path c:\data\rsop.xml</pre>

In this command, I’m calling the cmdlet to get RSoP data from a computer called win7-x86-1 and a user account on that computer called “cpandl\darren”. I store that data an XMLl file called c:\data\rsop.xml. Unlike the Get-GPOReport cmdlet, this RSoP cmdlet doesn’t allow you to leave out the –Path parameter and send the output to the pipeline, curiously. So if you want to load up your XML into a type-accelerated PowerShell variable like I did with Get-GPOReport, you’ll have to first save it to a file then load up that file into a variable as so:

<pre>$rsop = Get-Content c:\data\rsop.xml</pre>

Then you can navigate the XML nodes just as in my previous example.

### Living Outside the GP Module

There’s a lot of cool stuff you can do with the GP module, but there are also a fair number of limitations or things that it doesn’t do. To fill in the gaps, those of us at [SDM Software][2] and [GPOGUY.COM][3] have created a number of PowerShell-based GP utilities to help augment your toolkit, including:

  * At [www.sdmsoftware.com/freeware][1]:
  * <span style="text-decoration: underline;">GPMC Cmdlets 1.4</span>: Provides 25 cmdlets for pre-Windows 7/Windows Server 2008 R2 environments. Also includes some things that the GP Module does not—the ability to view GP links by GPO or SOM
  * <span style="text-decoration: underline;">GP Refresh Cmdlet</span>: Let’s you perform remote GP refreshes from PowerShell
  * <span style="text-decoration: underline;">GP Health Cmdlet</span>: Let’s you get GP processing health (including overall status, how long GP processing took, which GPOs were processed, etc.) from PowerShell

  * At [GPOGUY.COM][4] (Under the Free Tools Library)
  * <span style="text-decoration: underline;">Get-SDMGPOVersion</span> is like a PowerShell version of GPOTool—it checks AD & SYSVOL version number consistency across all GPOs
  * <span style="text-decoration: underline;">Invoke-SDMTouchGPO</span> provides a way of “touching” the version number on a GPO, thereby forcing clients to think it’s changed and perform a refresh of Group Policy

Our GPMC Cmdlets precede the existence of the Microsoft Group Policy module and provide GPMC functionality to PowerShell for versions of Windows prior to Windows 7 (e.g. Windows XP and Server 2003). You can also still use them under Windows 7, but many of the features are redundant to the Windows 7 GP module. However, as I mentioned above, there are some things you can do with our GPMC cmdlets that aren’t exposed by the GP module, including, most notably, getting information about GPO links. As an example, the **Get-SDMGPLink** cmdlet lets you report on GPO links, but more importantly, you can query GPO links by either scope (e.g. OU or domain) or GPO. For example, if I want to discover all the GPOs linked to the marketing OU, I can issue the following:

<pre>Get-SDMGPLink -Scope 'OU=Marketing,DC=cpandl,DC=com'</pre>

And the cmdlet will report all the GPOs linked to the Marketing OU, as shown in **Figure 5:**

![](/images/darren5.png)

You can also view where a particular GPO is linked by issuing the following command:

<pre>Get-SDMGPLink -Name 'Default Domain Policy'</pre>

### Group Policy Troubleshooting

Next up is the Group Policy health cmdlet, another one of the free cmdlets available at [www.sdmsoftware.com/freeware][1]. The health cmdlet is designed to quickly return GP health and processing information against one or more remote systems. It returns a red or green status about overall GP processing health and provides a lot more detail about the GPOs that were processed by a computer and user, what CSEs were processed and whether they succeeded or failed and other details such as whether loopback was enabled on the system, how long GP processing took and more. Once installed the cmdlet syntax is pretty straightforward—you can pass in a single computer name or a whole OU worth of computers and cmdlet will query the systems and return results to the pipeline. In addition, if you use the OutputbyXML parameter, the results will be returned as an XML document, which you can then store and navigate using PowerShell’s XML node navigation capabilities. An example of the output of the Health cmdlet is shown here:  

![](/images/darren6.png)

### Commercial Group Policy PowerShell Functionality

In this final section, I’ll detail the capabilities that you can get for managing GP using commercial solutions developed by **SDM Software**.  The first and most powerful of these is the [**Group Policy Automation Engine**][5]**.** The GPAE, first released in 2007, is the first and only product that provides the ability to automate read and writes to GPO settings, using PowerShell, of course. The GPAE supports about 80% of the existing policy areas, including Admin. Templates, Security policy, GP Preferences, Software Installation, Folder Redirection, and more. As a quick example, the following script lets you create new GP Preferences drive mapping policies based on input from a CSV file, complete with an item-level target that filters the drive mapping on a user group:

```
function Map-Drive
{
  param(
    [string]$DriveLetter,
    [string]$Share,
    [string]$Domain,
    [string]$GroupName
  )

  Write-Host "Writing Drive Mapping: $DriveLetter"
  $gpo = Get-SDMGPObject "gpo://cpandl.com/Drive Mapping Policy" -OpenbyName
  $path ='User Configuration/Preferences/Windows Settings/Drive Maps'
  $drives = $gpo.GetObject($path)

  $map = $drives.Settings.AddNew($DriveLetter)
  $map.Put('Action',[GPOSDK.EAction]'Create')
  $map.Put('Drive Letter',$DriveLetter)
  $map.Put('Location',$Share)
  $map.put('Reconnect', $true)
  $map.Put('Label as', $DriveLetter)


  # now do ILT
  $objUser = New-Object System.Security.Principal.NTAccount $Domain, $GroupName
  $strSID = $objUser.Translate([System.Security.Principal.SecurityIdentifier])
  $iilt = $GPO.CreateILTargetingList()
  $itm = $iilt.CreateIILTargeting([GPOSDK.Providers.ILTargetingType]'FilterGroup')
  $itm.put('Group',$groupName)
  $itm.put('UserInGroup',$true)
  $itm.put('SID',$strSID.Value)
  $iilt.Add($itm)

  # now add ILT to drive mapping and save the setting
  $map.Put('Item-level targeting',$iilt)
  $map.Save()
}

$driveInfo = Import-Csv -Path c:\data\drivemaps.csv
foreach ($drive in $driveInfo)
{
  Map-Drive -DriveLetter $drive.DriveLetter -Share $drive.Share `
            -Domain $drive.Domain -GroupName $drive.GroupName
}
```

The GPAE can also read settings out of GPOs and then use that information as input to perform other changes (e.g. reading settings out of one GPO and writing them to another one).

### Exporting and Comparing GPOs the PowerShell Way

To round out the commercial offering that SDM Software provides, the [GPO Reporting Pak][6] are GUI offerings that provide powerful reporting and comparison for GPOs. The Reporting Pak, composed of **GPO Compare** and **GPO Exporter** also provide PowerShell interfaces, so you can do cool stuff like use PowerShell to compare the settings of a GPO in one domain with a GPO in another, as shown in Figure 7.

![](/images/darren7.png)

### Summary

The bottom line is that regardless of whether you’re using the Microsoft Group Policy module, some of the free cmdlets available on SDM Software or GPOGUY.COM, or one of the commercial products listed here, there is now little that you can’t do with PowerShell and Group Policy!

[1]: http://www.sdmsoftware.com/freeware
[2]: http://www.sdmsoftware.com/
[3]: http://www.gpoguy.com/
[4]: http://gpoguy.com/
[5]: http://www.sdmsoftware.com/products/group-policy-automation-engine/
[6]: http://www.sdmsoftware.com/products/group-policy-reporting-pak/