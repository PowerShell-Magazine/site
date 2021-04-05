---
title: Chocolatey and PowerShell – Revolutionize Windows, Part II
author: Rob Reynolds
type: post
date: 2012-02-20T19:00:27+00:00
url: /2012/02/20/chocolatey-and-powershell-revolutionize-windows-part-ii/
views:
  - 17604
post_views_count:
  - 2068
categories:
  - Package Management
tags:
  - Chocolatey
  - Package Management

---
In the [first installment](/2012/02/15/chocolatey-and-powershell-revolutionize-windows-part-i/) we covered installing both a tool and an application. If the possibilities of Chocolatey have not yet started to solidify for you, hopefully this next segment will.

### Stay Up to Date

Chocolatey allows you to keep up to date with the update command. Do you remember how simple the command for install was from the last segment? Update is just as simple. You can run this with either ‘chocolatey update’ or ‘cup’ for short.

```powershell
PS> chocolatey update somePackageName
```

### One Update to Rule Them All

Let’s not forget the update command to rule them all. Think of it like windows update, but for everything else. This will update every package you have installed on your machine!

```powershell
PS> cup all
```

### Machine Environment Setup

Another possible scenario where Chocolatey and PowerShell can shine is surrounding a developer set up environment for hacking. When you pull down source code from time to time you will see notes that people have included to tell possible contributors what they need to do to set up the “environment.” Using Chocolatey and some PowerShell you can script that out and install everything they need without having to use some advanced code. For instance, take a look at [nuserve](https://github.com/davidalpert/nuserve#readme) on github. You run setup.cmd and it executes a PowerShell script that installs Chocolatey, NuGet, Ruby, DevKit, updates Gems, installs bundler, and then runs the builds. If you don’t think that is a powerful idea, consider that the author of nuserve, [David Alpert](http://blog.spinthemoose.com/), said the script was “all scripted to a one-liner. Genius.” For a second imagine having someone pull down your source and firm up any development discrepancies automatically before they even begin. They didn’t even have to read your readme.

### Under the Hood

Chocolatey uses some ingenuity with PowerShell on top of the already existing NuGet. It itself is about 500 lines of PowerShell code. I am not an expert at PowerShell, just someone who wanted to learn more about the language and hope that it sticks this time. Every time I use PS, I later forget almost everything I’ve learned. PowerShell is very powerful in what it can do &#8211; consider that you can actually put PowerShell text on the internet somewhere and run Invoke-Expression after literally downloading it to run a set of PowerShell commands. That’s really powerful when you think about it. The Chocolatey one line install takes advantage of that:

```powershell
PS> Invoke-Expression ((New-Object Net.WebClient).DownloadString('http://bit.ly/psChocInstall'))
```

Chocolatey puts some batch files on the path that call PowerShell and run the Chocolatey script, passing along the arguments. When Chocolatey runs, it uses NuGet to download packages and work on dependency resolution. Then it searches for and executes the chocolateyInstall.ps1 script if it finds it. It generates a runtime of PowerShell, importing helper modules for the script to use when executing. This allows the script to be very brief (more on this in **Creating Packages**).

It also looks for executables in the package folder, either there with the package or brought in during the execution of the chocolateyInstall script. When it finds these executables, it creates a batch command reference in a folder on the path so the commands are now available.

Chocolatey takes advantage of PowerShell’s ability to override commands. When you create packages and you call Write-Host or Write-Error, Chocolatey leverages that to note the information in a log before calling the underlying command. It really leverages the concept of _Splatting_ to do this.

```powershell
function Write-Host  {
   param(
      [Parameter(Position=0,ValueFromPipeline=$true)][object]$Object,
      [Parameter()][switch] $NoNewLine,
      [Parameter()][ConsoleColor] $ForegroundColor,
      [Parameter()][ConsoleColor] $BackgroundColor,
      [Parameter()][Object] $Separator
   )
   $chocoPath = Split-Path -Parent $helpersPath
   $chocoInstallLog = Join-Path $chocoPath 'chocolateyPSInstall.log'
   $Object | Out-File -FilePath $chocoInstallLog -Force -Append
   $oc = Get-Command Write-Host -Module Microsoft.PowerShell.Utility
   & $oc @PSBoundParameters
}
```


Notice in the command above how I grab the original command and then pass all of the parameters to it from my function with @PSBoundParameters? It takes my $PSBoundParameters variable and passes it to the function I am calling as parameters. This is great because it saves a lot of branching.

### Creating Packages

Remember how we first talked about how easy Chocolatey is to use? Creating packages is simple, too. You can literally create a package to install a full application with a nuspec (NuGet package specifications xml file) and a chocolateyInstall.ps1 file. The nuspec containing the information about the package will take you longer to gather and fill out compared to the time it’ll take you to prepare the download and install of an MSI in the chocolateyInstall.ps1 file. Chocolatey comes with helpers (<https://github.com/chocolatey/chocolatey/wiki/HelpersReference>) to shorten the work necessary in the chocolateyInstaller.ps1 file.

Imagine in your mind what it would take to download a file from a known location and then run in administrative mode to install it silently. Have you got a picture of that now? Now here’s how you install StExBar:

```powershell
PS> Install-ChocolateyPackage 'StExBar' 'msi' '/quiet' 'http://stexbar.googlecode.com/files/StExBar-1.8.3.msi' 'http://stexbar.googlecode.com/files/StExBar64-1.8.3.msi'
```

Notice the two urls? The optional second one is for 64bit if the application supports that. If there is not a 64bit version, just leave out the second url and Chocolatey handles it.

### Videos

There is a short video on YouTube demonstrating how easy it is to create a package. I create one for the [WinDirStat](http://windirstat.info/ )'s package. http://www.youtube.com/watch?v=Wt_unjS_SUo. You can also take a look at the [wiki](https://github.com/chocolatey/chocolatey/wiki/CreatePackages) for a more in depth explanation.

### Conclusion

Chocolatey is young, has very powerful concepts and a community that is rapidly growing. PowerShell drives it, giving Chocolatey the ability to quickly adapt to meet many needs. Plus, it uses NuGet as the packaging system, which brings the advantages of a packaging system that came before. Chocolatey supports an existing infrastructure of native installers and decentralized distributions using a centralized point of instructions. And because the chocolateyInstall.ps1 instructions are just PowerShell, they can be about virtually anything, not necessarily a call to download and install an MSI. Think of Chocolatey as a global automation tool for Windows.

This is not the first time someone has tried to bring the ideas of apt-get to Windows and it won’t be the last. Chocolatey has a good chance of succeeding where others have failed because simplicity is at its core. Chocolatey is simple to use, simple to create packages, and super simple to maintain packages.  The bar to entry is low because it builds on already familiar ideas and it leverages things that are already available with Windows.

Let’s get Chocolatey!
