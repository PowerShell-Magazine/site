---
title: Pimp Your Profile
author: Scott Muc
type: post
date: 2012-02-13T19:00:44+00:00
url: /2012/02/13/pimp-your-profile-2/
views:
  - 31321
post_views_count:
  - 7635
categories:
  - How To
tags:
  - How To

---
One characteristic of effective shell users is that they modify their environment in ways that makes them as efficient as possible. This concept is standard Unix-fu and now that PowerShell has introduced us to customizable profiles, we can enjoy the same luxury.

There are already [great resources](http://msdn.microsoft.com/en-us/library/windows/desktop/bb613488%28v=vs.85%29.aspx ) about PowerShell Profiles so I’m not going to repeat those details here. If you are curious about how profiles are implemented, then I recommend reading [Lee Holmes' post](http://www.leeholmes.com/blog/2006/04/26/the-story-behind-the-naming-and-location-of-powershell-profiles/ ) on some of the history behind the decision making.

The purpose of this article is to show you some of the things that you can do with a customized profile. I hope it inspires you to create your own and make it available for everyone to learn from. Here are a few things I have my profile doing for me:

### PowerShell Module auto-loader

```powershell
PS> Get-Module -ListAvailable |
Where-Object { $_.ModuleType -eq "Script" } | Import-Module
```

[PowerShell modules](http://msdn.microsoft.com/en-us/library/windows/desktop/dd878324%28v=vs.85%29.aspx ) are self-contained set of scripts that expose public functions as their interface. When modules are installed in one of your $env:PSModulePath locations, you can import them by name using Import-Module.  By default PowerShell doesn&#8217;t import any modules. I find this a bit irritating because I want modules that I install to be available to me whenever I&#8217;m in PowerShell. I&#8217;m probably influenced by Ruby and RubyGems here.

Since I haven&#8217;t found a reason why I wouldn&#8217;t want all my PowerShell modules loaded by default, I have a simple function that finds all script modules installed and imports them right way. The module I use frequently is [Pester](https://github.com/scottmuc/Pester ), a PowerShell module I’ve authored, to help test my PowerShell scripts. When I start a new shell, I have the following available right away:

```powershell
PS> Get-Module
ModuleType Name      ExportedCommands

Script     Pester    {It, Describe, New-Fixture, Invoke-Pester}
Script     posh-git  {GitTabExpansion, Get-GitDirectory, Get-GitStatus, tgit}
Script     PsGet     {Get-PsGetModuleInfo, Install-Module}
```

To manage my PowerShell Modules I use a tool called [PsGet](http://psget.net/ ) which is easy and light-weight.

### Modular function loader

```powershell
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
Resolve-Path $here\functions\*.ps1 |
	Where-Object { -not ($_.ProviderPath.Contains(*.Tests.*)) } | 
	ForEach-Object { . $_.ProviderPath }
```

This bit of code [dotsources](http://ss64.com/ps/source.html ) all ps1 files in my profile&#8217;s functions directory. I consciously chose to put re-usable functions in their own files in this directory. Here are a couple of the functions it imports:

  * `Edit-HostsFile` &#8211; pretty much does what it says. I can never remember where Windows keeps its hosts file, can you?
  * `Get-Bits` &#8211; returns the CPU architecture of your shell. Here’s this function in action:

```powershell
PS> Get-Bits
64-bit
```

### Prompt feedback

There&#8217;s a special function that you can put in your profile and it&#8217;s called &#8220;prompt&#8221;. It overrides the default prompt and allows you to specialize it the way you see fit. I found a [wonderful prompt](http://winterdom.com/2008/08/mypowershellprompt) that looks nice and provides information about the current directory. I’ve added some extra code in there to make it context aware of git directories, which is a feature of [PoshGit](https://github.com/dahlbyk/posh-git). I&#8217;ve also decided to put my prompt code in a prompt.ps1 file inside the directory that the modular function loader looks at.


![](/images/pimpprofile1.png)

### Functions, and aliases, and variables! Oh, my!

Here are some functions and customizations that I didn&#8217;t feel special enough to isolate on their own in the auto-loaded functions directory. This is where I feel the best place to put your specific customizations. For example, I put in a few shortcut functions that mimic UNIX commands that I use often.

  * which &#8211; tells me the location of where an executable lives
  * rm-rf &#8211; nukes a file or folder and all its contents
  * touch &#8211; creates an empty file (this is demoed in figure 1)
  * g &#8211; a shortcut for launching gvim

### Venture down a different PATH

```
$UserBinDir = "$($env:UserProfile)\bin"
$paths = @("$($env:Path)", $TransientScriptDir)
Get-ChildItem $UserBinDir | Where-Object { $paths += $_.FullName }
$env:Path = [String]::Join(&#8220;;&#8221;, $paths)
```

Almost always, I tend to favor augmenting my user/session state rather than the persistent machine state. This is why I have a bit of code in my profile after my PATH with some user specific customizations. This way I can be sure my changes aren’t affecting things at a service level.

One of these custom paths is the $TransientScriptDir. This directory is an area where I put scripts that I&#8217;ve downloaded for testing. The PowerShell community still relies on a lot of copy and paste distribution so when I find something worth trying I throw it into a file in this directory. Once it&#8217;s there I can then execute the script from any location. Here it is in action:

```powershell
PS> "Write-Host Hello World!"" | Out-File $TransientScriptDir\Say-Hello.ps1
PS> which Say-Hello.ps1
Definition
C:\Users\smuc\Documents\WindowspowerShell\scripts\Say-Hello.ps1
```



```powershell
PS> Say-Hello.ps1
Hello World!
```

The other additions to my PATH are every directory in %userprofile%\bin. Inside there I put stuff like curl, nuget, 7z, and other executable tools I find myself using on a day to day basis.

### Show it off

Lastly, there&#8217;s one thing I can&#8217;t stress enough&#8230; **keep your profile in source control**! There&#8217;s no reason for not keeping it somewhere like [github](https://github.com). If you take a look at https://github.com/scottmuc/poshfiles/  you can see that installation is as simple as a git clone and that&#8217;s it. As an added benefit, source control provides you is the ability to see what changes you&#8217;ve made.

### Summary

So, that&#8217;s a quick tour of some of the things that you can do with your PowerShell Profile. Please post your own personal customizations and reference them here. This is new territory in the Windows world and there&#8217;s a lot of new ground to break!
