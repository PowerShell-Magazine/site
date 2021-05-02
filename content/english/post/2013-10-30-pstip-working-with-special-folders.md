---
title: '#PSTip Working with Special Folders'
author: Shay Levy
type: post
date: 2013-10-30T18:00:00+00:00
url: /2013/10/30/pstip-working-with-special-folders/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Windows provides access to many folders of the current user via the SpecialFolders collection. A special folder can point to standard directories (e.g Favorites) or to a &#8220;virtual&#8221; folder (e.g My Computer). The SpecialFolders collection lists the identifiers (constants) and the contents (path) of each of the special folders in the SpecialFolders collection. The locations of these folders can have different values on different operating systems (and can be changed by the user).

The [Environment.SpecialFolder][1] enumeration specifies all constants used to retrieve directory paths to system special folders. Special folders are set by default by the system, or explicitly by the user, when installing a version of Windows. Your result may vary based on your OS and the installed .NET version.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Enum]::GetNames('System.Environment+SpecialFolder')
Desktop
Programs
MyDocuments
Personal
Favorites
Startup
Recent
SendTo
StartMenu
(...)
</pre>

Using the [Environment.GetFolderPath][2] static method we can get the physical path of a folder, pass an enumeration as an argument:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Environment]::GetFolderPath([System.Environment+SpecialFolder]::MyDocuments)
C:\Users\shay\Documents
</pre>

Or let PowerShell do the convertion for you by passing just the constant name:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Environment]::GetFolderPath('MyDocuments')
C:\Users\shay\Documents
</pre>

What if you don&#8217;t remember all constants? With the Get-SpecialFolder function you are worry free. Get-SpecialFolder provides an easy and convenient way to work with special folders. It lists all folders (or just a subset) that have a physical path and returns a custom object. All folder constants are properties of the resultant object and you can use IntelliSense to cycle through all of them.


    function Get-SpecialFolder
    {
        [CmdletBinding()]
        [OutputType('System.Management.Automation.PSCustomObject')]
        param( [string]$Name='*' )
    
        $pso = New-Object -TypeName PSObject
        $folders = [System.Enum]::GetNames('System.Environment+SpecialFolder')
    
        $folders | Where-Object {$_ -like $Name} | Sort-Object | ForEach-Object {
            if($folderPath = [System.Environment]::GetFolderPath($_))
            {
                $pso | Add-Member -MemberType NoteProperty -Name $_ -Value $folderPath
            }
        }
    
        $pso
    }
![](/images/SpecialFolders.png)

[1]: http://msdn.microsoft.com/en-us/library/system.environment.specialfolder.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.environment.getfolderpath.aspx