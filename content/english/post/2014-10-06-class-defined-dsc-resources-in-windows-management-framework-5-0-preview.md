---
title: Class-defined DSC resources in Windows Management Framework 5.0 Preview
author: Ravikanth C
type: post
date: 2014-10-06T16:00:16+00:00
url: /2014/10/06/class-defined-dsc-resources-in-windows-management-framework-5-0-preview/
views:
  - 32739
post_views_count:
  - 2866
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
By now, it should be no surprise that WMF 5.0 Preview added support for defining classes and user-defined data types. This is done using the formal syntax and semantics that you find in an object-oriented programming language. As per the September 2014 Preview release notes, the following are the supported scenarios:

  * Define DSC resources and their associated types by using the Windows PowerShell language.
  * Define custom types in Windows PowerShell by using familiar object-oriented programming constructs, such as classes, properties, methods, inheritance, etc.
  * Debug types by using the Windows PowerShell language.
  * Generate and handle exceptions by using formal mechanisms, and at the right level.

In this article, I will explain how to use classes in Windows PowerShell to define DSC resources. If you have written DSC resources using the PowerShell script module semantics, you will certainly appreciate how simple it is now to write and package a DSC resource.

  * You don&#8217;t have to create a schema MOF any more
  * A DSCResource folder inside the module folder is not needed anymore
  * You can now package multiple DSC resources in a single module script file

From a semantics point of view, the earlier script module needed three functions:

  1. Get-TargetResource
  2. Set-TargetResource
  3. Test-TargetResource

Within these three functions, we needed to define all key and required properties in each function. For example, the _Test-TargetResource_ and _Set-TargetResource_ functions must have the same number of parameters defined while _Get-TargetResource_ needs all the key and required properties. This is a lot of redundant code depending on how many properties you have.

Although the _xDSCResourceDesigner_ module helped generate lot of skeleton code including the schema MOF, many were still confused about the need for schema MOF and the exact syntax for these functions. This is where the new class-defined DSC resources come into picture. To this extent, WMF 5.0 added new language elements. In today&#8217;s article, we will see only what is relevant for the DSC resource demonstration.

The first one is the _Class_ keyword. This keyword is a true .NET type and all its members are public within the module scope. Here is a simple example. This just defines the class with no members.

```
class DSCDemo {
}
```

<a href="http://104.131.21.239/wp-content/uploads/2014/10/12.png" rel="lightbox[10554]"><img class="aligncenter size-full wp-image-10560" src="http://104.131.21.239/wp-content/uploads/2014/10/12.png" alt="1" width="534" height="215" srcset="http://www.powershellmagazine.com/wp-content/uploads/2014/10/12.png 534w, http://www.powershellmagazine.com/wp-content/uploads/2014/10/12-300x120.png 300w" sizes="(max-width: 534px) 100vw, 534px" /></a>

The second keyword is _Enum_. Using this, we can define new enumerations.

<pre class="brush: powershell; title: ; notranslate" title="">enum Test {
   Success = 0
   Failure = 1
   Unknown = 10
}
</pre>
<a href="http://104.131.21.239/wp-content/uploads/2014/10/21.png" rel="lightbox[10554]"><img class="aligncenter size-full wp-image-10563" src="http://104.131.21.239/wp-content/uploads/2014/10/21.png" alt="2" width="134" height="119" /></a>

The other language extensions for DSC resources include three attributes for defining resource properties.

  * _DscResource_ indicates that the class definition following the attribute is a DSC resource
  * _DscResourceKey_ indicates that the property is a DSC key property. This is equivalent to _Key_ qualifier in the legacy MOF schema files
  * _DscResourceMandatory_ indicates that the property is a required property. This is equivalent to _Required_ qualifier in the legacy MOF schema files.

With this knowledge, let us see how we can create a class-defined DSC resource. The following code snippet shows a skeleton for a DSC resource.

```
enum Ensure
{
   Absent
   Present
}

[DscResource()]
class ResourceName
{
   [DscResourceKey()]
   #Some Key Property

   [DscResourceMandatory()]
   #Some Mandatory property

   #Use of enumeration
   [Ensure] $Ensure

   #Set function similar to Set-TargetResource
   [void] Set()
   {
      #Set logic goes here
   }

   #Test function similar to Test-TargetResource
   [bool] Test()
   {
      #Test logic goes here
   }

   #Get function similar to Get-TargetResource
   [Hashtable] Get()
   {
      #Get logic goes here
   }
}
```

As you see above, the _Get_, _Set_, and _Test_ functions map to _Get-TargetResource_, _Set-TargetResource_, and _Test-TargetResource_ functions. The execution flow of the DSC resource itself did not change. _Test_ function gets executed first and based on its return value (_True_ or _False_), _Set_ either gets skipped or executed. The _Get_ function of a resource is used when the _Get-DscConfiguration_ cmdlet is called.

Now, let us build a functional resource from this skeleton. For the demonstration, I will show you the class-defined resource I built for managing _hosts_ file on Windows.

    enum Ensure
    {
       Absent
       Present
    }
    
    [DscResource()]
    class HostsFile
    {
       [DscResourceKey()]
       [string] $IPAddress
    
       [DscResourceKey()]
       [string] $HostName
    
       [Ensure] $Ensure
    
       [void] Set()
       {
          $hostEntry = "`n${IPAddress}`t${HostName}"
          if($Ensure -eq [Ensure]::Present)
          {
              Write-Verbose "Adding a Hosts File entry"
              Add-Content -Path "${env:windir}\system32\drivers\etc\hosts" -Value $hostEntry -Force -Encoding ASCII
              Write-Verbose "Added a hosts File entry"
          }
          else
          {
              Write-Verbose "removing hosts file entry"
              ((Get-Content "${env:windir}\system32\drivers\etc\hosts") -notmatch "^\s*$") -notmatch "^[^#]*$IPAddress\s+$HostName" | Set-Content "${env:windir}\system32\drivers\etc\hosts"
              Write-Verbose "removed hosts file entry"
          }
        }
    [bool] Test()
    {
        try {
           Write-Verbose "Checking if hosts file exists"
           $entryExist = ((Get-Content "${env:windir}\system32\drivers\etc\hosts") -match "^[^#]*$IPAddress\s+$HostName")
    
           if ($Ensure -eq "Present") {
               if ($entryExist) {
                   Write-Verbose "Hosts file entry does not exist"
                   return $true
               } else {
                   Write-Verbose "Hosts file entry does not exist while it should"
                   return $false
               }
           } else {
               if ($entryExist) {
                   Write-Verbose "Hosts file entry exists while it should not"
                   return $false
               } else {
                   Write-Verbose "Hosts file entry does not exist"
                   return $true
               }
            }
         }
         catch {
             $exception = $_
             Write-Verbose "Error occurred"
             while ($exception.InnerException -ne $null)
             {
                 $exception = $exception.InnerException
                 Write-Verbose $exception.message
             }
          }
      }
    
      [HostsFile] Get()
      {
          $Configuration = [hashtable]::new()
          $Configuration.Add("IPAddress",$IPAddress)
          $Configuration.Add("HostName",$HostName)
    
          Write-Verbose "Checking Hosts file entry"
          try {
             if ((Get-Content "${env:windir}\system32\drivers\etc\hosts") -match "^[^#]*$IPAddress\s+$HostName") {
                Write-Verbose "Hosts file entry found"
                $Configuration.Add('Ensure','Present')
             } else {
                Write-Verbose "Hosts File entry not found"
                $Configuration.Add('Ensure','Absent')
             }
          }
    
          catch {
             $exception = $_
             Write-Verbose "Error occurred"
             while ($exception.InnerException -ne $null)
             {
                 $exception = $exception.InnerException
                 Write-Verbose $exception.message
             }
          }
          return $Configuration
     }
     }
Now, all we need to do is copy the above code into a .PSM1 file and generate the module manifest using the _New-ModuleManifest_ cmdlet. Once you are done, you can copy both these files into a folder with the same name as .PSM1 file under the module path. On my system, I saved these files as HostsFile.PSM1 and HostsFile.PSD1 and stored them under _C:\Program Files\WindowsPowerShell\Modules_.

In the current implementation (September 2014 preview), the class-defined resources do not appear in the _Get-DscResource_ cmdlet output. This is not implemented yet. However, you can use this resource in a configuration script by using _Import-DscResource_. Here is a sample configuration script.


    Configuration Demo {
        Import-DscResource -Module HostsFile
        HostsFile Demo {
           IPAddress = "10.10.10.1"
           HostName = "TestHost10"
           Ensure = "Present"
        }
    
        HostsFile Demo1 {
           IPAddress = "10.10.10.110"
           HostName = "TestHost110"
           Ensure = "Present"
        }
    }
    Demo
This is it. We have a new class-defined DSC resource. If you want to add one more such resource to the same PSM1 file, you can simply add another class block and follow the same semantics as above. There is more than just creating this simple resource file. We will look into the other details of class-defined resources in a later post.