---
title: Creating a Simple DSC Resource
author: Jan Egil Ring
type: post
date: 2015-07-02T17:19:33+00:00
url: /2015/07/02/creating-a-simple-dsc-resource/
views:
  - 16325
post_views_count:
  - 3911
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In this article, we will look at how to get started creating a very simple DSC resource.

Creating DSC resource is, by many, considered an advanced topic in PowerShell. While that, of course, is true for more complex resources, the purpose of this article is to show that it also can be done relatively easy.

### Background – what are we trying to accomplish?

The purpose of the new resource we will create in this article is to manage Data Collector Sets in Performance Monitor. Data Collector Sets is a collection of performance counters that can be scheduled to run on a regular basis, or started manually.

Here is an example from a Windows Server 2012 R2 machine, showing the counters available in the default “Server Manager Performance Monitor” introduced in Windows Server 2012:

![](/images/dscrescreate1.png)

Data settings for each Data Collector Set can be managed in the Data Manager:

![](/images/dscrescreate2.png)

Several settings such as Minimum free disk can be configured by using Data Manager:

![](/images/dscrescreate3.png)

It is also possible to configure Actions in order to perform tasks like copying the Performance data-gathered by the collector to an alternate location, such as a central file server. In order to manage data growth, it is also possible to configure Actions for deleting files and reports older than a certain age:

![](/images/dscrescreate4.png)

In the properties of the Data Collector Set, basic settings such as schedules and stop conditions can be managed:

![](/images/dscrescreate5.png)

That was a lot of settings. Shouldn’t the DSC resource for managing Data Collector Sets be “simple”? It will be, due to the use of templates. A Data Collector Set can be saved as a template in XML format:

![](/images/dscrescreate6.png)

Importing and exporting these templates can be performed by using the command-line utility logman.exe:

![](/images/dscrescreate7.png)

This means that templates can be managed by using a tool which is familiar to people who is used to working with Performance Counters, while it will be much easier to create a DSC resource for this purpose.

The task for our new DSC resource will simply be to create a given Data Collector Set if it does not already exist, based on a template file.

Bonus tip: The [PAL (Performance Analysis of Logs) tool][1] is a powerful tool that reads in a performance monitor counter log and analyzes it using known thresholds. It contains template files for most of the major Microsoft products such as Active Directory, IIS, MOSS, SQL Server, BizTalk, Exchange, as well as 3<sup>rd</sup> party products such as Citrix XenApp and VMware View. These template files can be exported as Performance Monitor template files:

![](/images/dscrescreate8.png)

This means there is a lot of template files available for our new DSC resource as part of the PAL tool:

![](/images/dscrescreate9.png)

I would recommend you to use these templates as a foundation, import them on a temporary machine, adjust settings such as schedules and actions for uploaded log files to a central location, and then export the customized template.

The PAL tool itself is based on PowerShell, so you could also automate the process of analyzing the log files copied to the central location:

![](/images/dscrescreate10.png)

Sample output from a PAL Report:

![](/images/dscrescreate11.png)

A full sample report can be found [here][2].

**Creating the Logman DSC Resource**

Now that we have a full understanding of what we are trying to accomplish, we can proceed to create the actual DSC resource. Since its purpose is to ensure that a specific Data Collector Set is present, a natural name for the resource would be “Logman”. Before proceeding to the technical details, it’s a good idea to figure out what properties should be available. In this case we would need at least two properties:

-DataCollectorSetName: The name of the Data Set Collector. This will be the key property, meaning that this property will uniquely identify a target resource – in this case a Data Set Collector.

-XmlTemplatePath – The path (either a central UNC path or a local path) to the template exported from Performance Monitor

-Ensure – This is not required, but I’ve chosen to include it in order to be able to remove obsolete Data Collector Sets

This means that our goal is to create a DSC resource which we can use like this:

```powershell
Logman Hyper-V {
    DataCollectorSetName = 'Hyper-V'
    Ensure = 'Present'
    XmlTemplatePath = 'C:\PerfLogs\Templates\HyperV.xml'
} 
```

Instead of creating everything by hand, we are going to leverage the DSC Resource Designer Tool created by the PowerShell team. If you are running WMF 5.0, you can leverage PowerShellGet to download and install the xDSCResourceDesigner module from the PowerShell Gallery:

```powershell
Find-Module -Name xDSCResourceDesigner | Install-Module -Force
```

If you are running PowerShell 4.0, which is the minimum version required for authoring DSC configurations and resources), you can download the latest version from the [xDSCResourceDesigner repository][3] on GitHub.

When the module is installed, we can list the available commands using Get-Command:

![](/images/dscrescreate12.png)

In our scenario, we will use New-xDscResourceProperty to define the properties we need (DataCollectorSetName, Ensure, and  XmlTemplatePath). And then, we will use New-xDscResource to create the resource.

Before we do that we will need to create a PowerShell module to create the new resource into.

```powershell
New-Item -Path "$env:ProgramFiles\WindowsPowerShell\Modules" -Name PSCommunityDSCResources -ItemType Directory
```

The minimum requirement for the new module is having a module manifest, as the version number specified in the manifest also will be the version of the DSC resource.

```powershell
New-ModuleManifest -Path "$env:ProgramFiles\WindowsPowerShell\Modules\PSCommunityDSCResources\PSCommunityDSCResources.psd1" -Guid (([guid]::NewGuid()).Guid) -Author 'Jan Egil Ring' -CompanyName PSCommunity -ModuleVersion 1.0 -Description 'Example DSC Resource Module for PSCommunity' -PowerShellVersion 4.0 -FunctionsToExport '*.TargetResource'
```

Next, we can go ahead and create the DSC resource by using the two mentioned commands from the xDSCResourceDesigner module:

```powershell
# Define DSC resource properties 

$DataCollectorSetName = New-xDscResourceProperty -Type String -Name DataCollectorSetName -Attribute Key
$Ensure = New-xDscResourceProperty -Name Ensure -Type String -Attribute Write -ValidateSet "Present", "Absent"
$XmlTemplatePath = New-xDscResourceProperty -Name XmlTemplatePath -Type String -Attribute Required

# Create the DSC resource 
New-xDscResource -Name Logman -Property $DataCollectorSet,$Ensure,$XmlTemplatePath -Path "$env:ProgramFiles\WindowsPowerShell\Modules\PSCommunityDSCResources" -ClassVersion 1.0 -FriendlyName Logman –Force
```

We should now have the following file and folder structure in the module folder:

![](/images/dscrescreate13.png)

The Logman.schema.mof file defines the 3 parameters we defined when creating the resource, and no further changes to this file is required.

The Logman.psm1 file contains a ready-to-use template for the 3 required PowerShell functions Get-, Set-, and Test-TargetResource.

You can see the complete template file in the middle of [this article][4] on the PowerShell Team blog, where you also can find other details about using the xDSCResourceDesigner module.

Our task is now to edit the three functions in the Logman.psm1 file so they perform the required tasks.

Let us start with Get-TargetResource, which should return a hash table containing the three properties for our resource. Since logman is a text-based command-line tool, we will need to parse the information we are interested in. In this scenario we want the name of all Data Collector Sets, and this use Select-String and the –replace operator to extract this information. If the query contains the value of the $DataCollectorSetName variable, we will set the value of $Ensure to $true. We then create a hash table containing the 3 properties, which is then returned as output from the function:

```powershell
$logmanquery = (logman.exe query $DataCollectorSetName | Select-String -Pattern Name) -replace 'Name:                 ', ''
if ($logmanquery -contains $DataCollectorSetName)
{
$Ensure = $true
}
else
{
$Ensure = $false
}

$returnValue = @{
DataCollectorSetName = $DataCollectorSetName
Ensure               = $Ensure
XmlTemplatePath      = $XmlTemplatePath
}
$returnValue
```

Note that we do not calculate or gather the value of $XmlTemplatePath, we simply use the value which is specified by the DSC configuration author.

Next is Set-TargetResource; this function should contain the necessary logic to put the target resource (in our case a Data Collector Set) into the desired state. It should not return any output, but Debug and/or Verbose messages is recommended in order to make it easier to see what is happening during a consistency check. Since we decided to implement the Ensure property, we will also need to create the necessary logic based on whether the value of Ensure is Present or Absent:

```powershell
if( $Ensure -eq 'Present' )
  {
    if (Test-Path -Path $XmlTemplatePath)
    {
      Write-Verbose -Message "Importing logman Data Collector Set $DataCollectorSetName from Xml template $XmlTemplatePath"
    
  	  $null = logman.exe import -n $DataCollectorSetName -xml $XmlTemplatePath
    } else
    {
      Write-Verbose -Message "$XmlTemplatePath not found or temporary inaccessible, trying again on next consistency check"

    }
  }
  elseif( $Ensure -eq 'Absent' )
  {
    Write-Verbose -Message "Removing logman Data Collector Set $DataCollectorSetName"
    $null = logman.exe delete $DataCollectorSetName
  }
```

Last is Test-TargetResource which should return a Boolean $true or $false. We reuse the same logman query as used in Get-TargetResource and returns the appropriate the Boolean based on the value of $Ensure and whether the Data Collector Set exists or not:

```powershell
$logmanquery = (logman.exe query $DataCollectorSetName | Select-String -Pattern Name) -replace 'Name:                 ', ''

if ($logmanquery -contains $DataCollectorSetName)
  {
    Write-Verbose -Message "Data Collector $DataCollectorSetName exists"

    if( $Ensure -eq 'Present' )
    {
      return $true
    }
    elseif ( $Ensure -eq 'Absent' )
    {
      return $false
    }
  }
  else
  {

    Write-Verbose -Message "Data Collector $DataCollectorSetName does not exist"

    if( $Ensure -eq 'Present' )
    {
      return $false
    }
    elseif ( $Ensure -eq 'Absent' )
    {
      return $true
    }
  }
```

Due to the implementation of Ensure, the code might be a bit more complicated than necessary. However, this shouldn\`t be too hard if you have some experience with scripting constructs in PowerShell.

At this point, we are ready to test our new resource. By running Get-DscResource we can see whether it is registered properly as a valid resource. Highlighted in the following screenshot, it looks fine:

![](/images/dscrescreate14.png)

Let’s start by creating a new configuration. Since this is a custom DSC resource, we need to use Import-DscResource to import it in order for the DSC engine to see it.

After creating a node block, we specify the name of our resource, Logman:

![](/images/dscrescreate15.png)

In the above example, Ctrl+Space is used to get IntelliSense for valid properties. Note that the PsDscRunAsCredential property is only available in PowerShell 5.0 and later.

Here is a complete example of using the new resource:

```powershell
Configuration LogmanHyperV {
	Import-DscResource -ModuleName PSCommunityDSCResources
    node localhost {
        Logman Hyper-V {
        DataCollectorSetName = 'Hyper-V'
        Ensure = 'Present'
        XmlTemplatePath = '\\domain.local\IT-Ops\Perfmon-templates\HyperV.xml'
        }
    }
} 

LogmanHyperV -OutputPath c:\temp 
Start-DscConfiguration -ComputerName localhost -Path C:\temp -Wait -Verbose
```

After compiling it to a MOF-file and pushing it to the target node using Start-DscConfiguration, we can see the Verbose messages we added to the resource:

![](/images/dscrescreatefinal9.png)

We could open Performance Monitor to see if the Data Collector Set is present, but it’s also possible to simply run logman to verify:

![](/images/dscrescreatefinal8.png)

It is important to verify that all aspects of the resource works, thus the following example is using the same configuration as above – but Ensure is now changed to “Absent”:

![](/images/dscrescreatefinal7.png)

Again, we can use logman to verify that the Data Collector Set was removed.

![](/images/dscrescreatefinal6.png)

The complete module containing the DSC resource is available in [this][5] GitHub repository.

As you might notice, I’ve also added a few more items to the module not mentioned above. Specifically the folders Examples, ResourceDesignerScripts, and Tests:

![](/images/dscrescreatefinal5.png)

These are all based on best practices stated in the article “[PowerShell DSC Resource Design and Testing Checklist][6]” written by the PowerShell Team and won’t be covered in detail in this article.

I would also like to highlight [Logman\_Example\_2.ps1][7], which shows how our custom resource can be combined with the built-in resources:

![](/images/dscrescreatefinal4.png)

One feature that I considered adding to the Logman DSC resource was the ability to ensure that the latest version of an XML file was applied. Since that would complicate the *-TargetResource functions I skipped that from the resource itself to keep it simple.

In the [Logman\_Example\_2.ps1][7] I also show an example of using the built-in Script resource in order to re-create the Data Collector Set if the modified date of the central file is newer than the local copy.

However, I do find the use of the Script resource quite ugly, and I would rather suggest you use versioning in the Data Collector Set name in order to deploy updated versions of the XML file. Then you could set the current version to Ensure=’Absent’, and the new version to Ensure=’Present’:

![](/images/dscrescreatefinal3.png)

A couple of things which should be added to the Logman resource to make it complete is [Pester][8] tests and error handling. As stated earlier the goal was to show creating a simple DSC resource, and thus those features won’t be covered in this article.

### Creating the Logman DSC resource as a class-based resource

In PowerShell 5.0, the process of creating custom DSC resources is drastically simplified due to the use of classes. This means that we no longer need to create a .schema.mof file. The folder structure is also simplified.

Now we can create the PowerShell module for the new DSC resource in 3 lines:

```powershell
New-Item -Path "$env:ProgramFiles\WindowsPowerShell\Modules" -Name PSCommunityDSCClassBasedResources -ItemType Directory

New-ModuleManifest -Path "$env:ProgramFiles\WindowsPowerShell\Modules\PSCommunityDSCClassBasedResources\PSCommunityDSCClassBasedResources.psd1" -Guid (New-Guid).Guid -Author 'Jan Egil Ring' -CompanyName PSCommunity -ModuleVersion 1.0 -Description 'Example class based DSC Resource module for PSCommunity' -PowerShellVersion 5.0 -DscResourcesToExport * -RootModule PSCommunityDSCClassBasedResources.psm1

New-Item -Path "$env:ProgramFiles\WindowsPowerShell\Modules\PSCommunityDSCClassBasedResources" -Name PSCommunityDSCClassBasedResources.psm1 -ItemType File
```

This gives us the following file and folder structure:

![](/images/dscrescreatefinal2.png)

In the .psm1 file we define a new class using the class keyword and DscResource declaration introduced in PowerShell 5.0. We then define the properties we previously had to declare in a separate schema.mof file directly inside the new class. Get/Set/Test-TargetResource is replaced by class methods:

![](/images/dscrescreatefinal1.png)

Now that we have the skeleton ready, we can simply copy the code containing the logic for Get/Set/Test from the legacy DSC resource.

In class-based resources, we refer to the current instance of a class by using the $this variable. If we need to reference a parameter supplied to the Get/Set/Test methods, we will need to reference them using $this.variablename, not just $variablename. That is the most important thing to edit when copying code from legacy DSC resources. Here is an example where for example $Ensure is changed to $this.Ensure:

![](/images/dscrescreatefinal.png)

The rewritten class-based version of the Logman DSC resource is available in [this repository][9] on GitHub, as well as on the [PowerShell Gallery][10]. To install the module from the PowerShell Gallery, use Install-Module:

```powershell
Install-Module -Name Logman
```


You can find more details about class based resources in the article [Writing a custom DSC resource with PowerShell classes][11] on Microsoft TechNet.

### Summary

By leveraging the new resource we have created, it should be an easy task to establish baselines for collecting performance data.

Actions in the Data Collector Set’s Data Manager makes it possible to automatically purge and upload log files to a central UNC path.

It is also possible to automate analysis of the centrally uploaded log files by using the PAL (Performance Analysis of Logs) tool. This tool also provides many templates for monitoring specific server roles and products such as Hyper-V and VMware.

This concludes our walkthrough on how to create a simple DSC resource. We have looked at how to accomplish the task using both the legacy method in PowerShell 4.0, as well as the new and simplified class-based method introduced in PowerShell 5.0.

[1]: http://pal.codeplex.com
[2]: http://1drv.ms/1FvVQAR
[3]: https://github.com/PowerShell/xDSCResourceDesigner
[4]: http://blogs.msdn.com/b/powershell/archive/2013/11/19/resource-designer-tool-a-walkthrough-writing-a-dsc-resource.aspx
[5]: https://github.com/janegilring/PSCommunity/tree/master/DSC/PSCommunityDSCResources
[6]: http://blogs.msdn.com/b/powershell/archive/2014/11/18/powershell-dsc-resource-design-and-testing-checklist.aspx
[7]: https://github.com/janegilring/PSCommunity/blob/master/DSC/PSCommunityDSCResources/DSCResources/Examples/Logman_Example_2.ps1
[8]: /2014/03/12/get-started-with-pester-powershell-unit-testing-framework/
[9]: https://github.com/janegilring/PSCommunity/blob/master/DSC/PSCommunityDSCClassBasedResources/PSCommunityDSCClassBasedResources.psm1
[10]: https://www.powershellgallery.com/packages/Logman/
[11]: https://technet.microsoft.com/en-us/library/dn948461.aspx