---
title: Measuring PowerShell scripts
author: Shay Levy
type: post
date: 2013-05-13T16:00:00+00:00
url: /2013/05/13/measuring-powershell-scripts/
categories:
  - How To
tags:
  - How To

---
Earlier this year, Microsoft released <a href="http://104.131.21.239/2013/03/20/the-windows-powershell-3-0-sdk-sample-pack" target="_blank">the Windows PowerShell 3.0 SDK Sample Pack</a> which includes a lot of code samples that show how to build applications based on Windows PowerShell 3.0. If you browsed through the samples, you probably noticed that all samples are written in C# and require Visual Studio 2010 to build and compile the samples. That&#8217;s all good for developers, but what about IT Pros?

One of the samples, <a href="http://code.msdn.microsoft.com/Script-Line-Profiler-Sample-80380291" target="_blank">the Script Line Profiler Sample</a>, shows how to create a script line profiler using the new Windows PowerShell 3.0 Abstract Syntax Tree (AST) support. Out-of-the-box, PowerShell offers two Measure cmdlets: _Measure-Command_ and _Measure-Object_. The former allows you to measure the time it takes to run script blocks and cmdlets and the latter enables us to calculate the numeric properties of objects, and more.

The Script Line Profiler Sample provides an additional measuring cmdlet, the _Measure-Script_ cmdlet. With _Measure-Script_ we can measure the execution time of each script statement (line). We can identify code bottlenecks and pin point parts of code that take longer to execute than expected. That&#8217;s surely a cmdlet you&#8217;d want to have in your utility belt!

Anyway, as stated above, there is just one MAJOR <a href="http://www.urbandictionary.com/define.php?term=pita%20%28pain%20in%20the%20ass%29" target="_blank">PITA</a> &#8211; it requires Visual Studio! In this article, I want to show a way to compile the project without having to install VS, just by using cmdlets available in PowerShell 3.0. First, let&#8217;s browse the sample files. You can do that by clicking on the <a href="http://code.msdn.microsoft.com/Script-Line-Profiler-Sample-80380291/view/SourceCode#content" target="_blank">Browse Code tab</a>.

![](/images/psprofiler01.png)

You can see that the sample includes a few files, the main code required for the sample to work is in the _PSProfiler.cs_ file . So, here&#8217;s the plan. We need to download that file, get the source code, compile it, add the compiled DLL to our PowerShell session, run _Import-Module_ with the full path to the sample DLL, and then run the _Measure-Script_ cmdlet on a script file.

A lot of work, right? Let&#8217;s see how we can automate the process:

1. Use the _Invoke-WebRequest_ cmdlet to read the source code of PSProfiler.cs
  
2. Extract the source code from the PRE tag.
  
3. Compile it; no need to have VS, we can use the _Add-Type_ cmdlet.
  
4. Import the compiled DLL using _Import-Module_
  
5. Use the _Measure-Script_ cmdlet.

Here&#8217;s the code snippet that does all of that:

```
# the url of the PSProfiler.cs file
$url='http://code.msdn.microsoft.com/Script-Line-Profiler-Sample-80380291/sourcecode?fileId=70887&pathId=217486489'
$iwr = Invoke-WebRequest -Uri $url

# the source code is contained in the PRE tag
# you can see this by investigating the page source in your browser
$code = @($iwr.ParsedHtml.getElementsByTagName('PRE')).innerText

# compile the code and output it as a .NET assembly
Add-Type -TypeDefinition $code -OutputAssembly .\PSProfiler.dll

# import it
Import-Module .\PSProfiler.dll -Verbose
VERBOSE: Loading module from path 'D:\temp\PSProfiler.dll'.
VERBOSE: Importing cmdlet 'Measure-Script'.

#verify the module exists
Get-Module PSProfiler

# explore the command
Get-Command Measure-Script
```

Measure-Script can operate in two ways: profile a script file or create a script line profiler AST object. The following sample script will be used as a test script for both uses. It reads a list of user names (5 in this example) from a CSV file and, calculates each user&#8217;s home directory size and create a new custom object for each user.

```
############
## c:\script.ps1
Import-Csv D:\temp\users.csv | Foreach-Object{
    $user = Get-ADUser $_.SamAccountName -Properties HomeDirectory
    $homeDir = Get-ChildItem $_.HomeDirectory -Recurse -Force | Measure-Object Length -Sum

    New-Object PSObject -Property @{
        UserName = $_.SamAccountName
        HomeDirectorySizeInMB = '{0:N2}' -f ($homeDir/1mb)
    }
}
```

Let&#8217;s measure its performance.


	PS> Measure-Script -Path c:\script.ps1
	Time Line
	---- ----
	8643 Import-Csv D:\temp\users.csv | Foreach-Object{
	   0
	  37     $user = Get-ADUser $_.SamAccountName -Properties HomeDirectory
	8597     $homeDir = Get-ChildItem $user.HomeDirectory -Recurse -Force | Measure-Object Length -Sum
	   0
	   6     New-Object PSObject -Property @{
	   0         UserName = $_.SamAccountName
	   0         HomeDirectorySizeInMB = '{0:N2}' -f ($homeDir.Sum/1mb)
	   0     }
	   0 }
The output object is made of two properties, _Time_, measures statement execution time for each line in script (in milliseconds), and _Line_ which is the line code being measured. The second produces an AST object. You can start digging into the object and start experimenting with AST and get to know its members.


	PS> $ast = Measure-Script -Path c:\script.ps1 -Ast
	PS> $ast | Get-Member
	   TypeName: System.Management.Automation.Language.ScriptBlockAst
	
	Name               MemberType Definition
	----               ---------- ----------
	Equals             Method     bool Equals(System.Object obj)
	Find               Method     System.Management.Automation.Language.Ast Find(System.Func[System.Management.Automatio...
	FindAll            Method     System.Collections.Generic.IEnumerable[System.Management.Automation.Language.Ast] Find...
	GetHashCode        Method     int GetHashCode()
	GetHelpContent     Method     System.Management.Automation.Language.CommentHelpInfo GetHelpContent()
	GetScriptBlock     Method     scriptblock GetScriptBlock()
	GetType            Method     type GetType()
	ToString           Method     string ToString()
	Visit              Method     System.Object Visit(System.Management.Automation.Language.ICustomAstVisitor astVisitor...
	BeginBlock         Property   System.Management.Automation.Language.NamedBlockAst BeginBlock {get;}
	DynamicParamBlock  Property   System.Management.Automation.Language.NamedBlockAst DynamicParamBlock {get;}
	EndBlock           Property   System.Management.Automation.Language.NamedBlockAst EndBlock {get;}
	Extent             Property   System.Management.Automation.Language.IScriptExtent Extent {get;}
	ParamBlock         Property   System.Management.Automation.Language.ParamBlockAst ParamBlock {get;}
	Parent             Property   System.Management.Automation.Language.Ast Parent {get;}
	ProcessBlock       Property   System.Management.Automation.Language.NamedBlockAst ProcessBlock {get;}
	ScriptRequirements Property   System.Management.Automation.Language.ScriptRequirements ScriptRequirements {get;}
	
	PS&gt; $ast
	
	ParamBlock         :
	BeginBlock         :
	ProcessBlock       :
	EndBlock           : Import-Csv D:\temp\users.csv | Foreach-Object{
	
				 $user = Get-ADUser $_.SamAccountName -Properties HomeDirectory
				 $homeDir = Get-ChildItem $user.HomeDirectory -Recurse -Force | Measure-Object Length -Sum
	
				 New-Object PSObject -Property @{
				     UserName = $_.SamAccountName
				     HomeDirectorySizeInMB = '{0:N2}' -f ($homeDir.Sum/1mb)
				 }
			     }
	DynamicParamBlock  :
	ScriptRequirements :
	Extent             : Import-Csv D:\temp\users.csv | Foreach-Object{
	
				 $user = Get-ADUser $_.SamAccountName -Properties HomeDirectory
				 $homeDir = Get-ChildItem $user.HomeDirectory -Recurse -Force | Measure-Object Length -Sum
	
				 New-Object PSObject -Property @{
				     UserName = $_.SamAccountName
				     HomeDirectorySizeInMB = '{0:N2}' -f ($homeDir.Sum/1mb)
				 }
			     }
	
	Parent             :

Finally, we don&#8217;t want to hit the web and compile the project each time we need to use _Measure-Script_, it would be better to compile it once and create a local module.

```
$url='http://code.msdn.microsoft.com/Script-Line-Profiler-Sample-80380291/sourcecode?fileId=70887&pathId=217486489'
$iwr = Invoke-WebRequest -Uri $url
$code = @($iwr.ParsedHtml.getElementsByTagName('PRE')).innerText

# get the local user module folder
$UserModulesFolder = "$env:USERPROFILE\Documents\WindowsPowerShell\Modules"

# create new folder for the module
$profiler = New-Item -Path $UserModulesFolder -Name PSProfiler -ItemType Directory

# generate the DLL
Add-Type -TypeDefinition $code -OutputAssembly "$profiler\PSProfiler.dll"

# PDB files are generated when you build a VS project.
#They contain information relating to the built binaries which VS can interpret.
#They are not necessary for our DLL so we remove them.
Get-ChildItem $profiler -Filter *.pdb | Remove-Item -Force

# create new manifest file
New-ModuleManifest -Path $profiler\PSProfiler.psd1 -RootModule "PSProfiler.dll" -Description 'Script line profiler using Windows PowerShell 3.0 AST' -PowerShellVersion 3.0 -DotNetFrameworkVersion 4.0

# verify that module is discoverable
PS> Get-Module -ListAvailable PSProfiler
    Directory: C:\Users\Shay\Documents\WindowsPowerShell\Modules

ModuleType Name                                ExportedCommands
---------- ----                                ----------------
Binary     PSProfiler                          Measure-Script
```

You may also want to check Adam Driscoll&#8217;s work on the subject. He wrote about Advanced Script Profiling [HERE][1] , and on [his blog][2].

[1]: /2011/11/15/advanced-script-profiling
[2]: http://csharpening.net/?p=1226