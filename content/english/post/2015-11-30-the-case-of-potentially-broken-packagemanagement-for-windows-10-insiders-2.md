---
title: The case of potentially broken Package Management for Windows 10 Insiders
author: Ben Gelens
type: post
date: 2015-11-30T17:00:33+00:00
url: /2015/11/30/the-case-of-potentially-broken-packagemanagement-for-windows-10-insiders-2/
views:
  - 14784
post_views_count:
  - 2281
categories:
  - Package Management
tags:
  - Package Management

---
I’ve been running in the Windows Insiders fast ring since it was incepted and went from Windows 8.1 to Windows 10 November update with all the insider builds in between. I must say, Windows upgrades have never felt so stable and this has proven to be the fastest way to the newest WMF 5.0 builds.

A couple of months ago though, I stumbled upon a curious issue where my PackageManagement module misbehaved after a fast ring update. Find-Module would complain about a missing PSModule provider and I wasn’t able to do anything anymore in regard to interacting with the PowerShell Gallery. I&#8217;ve investigated and finally contacted the PowerShell team. We found out I got an issue which was very unlikely to occur and after we fixed it we decided not to publish a blog because of the unlikelihood of the occurrence. Now, a couple of months later and on Windows 10 November update, I’ve found a couple of others who&#8217;ve experienced my issue as well. Hence this blog post.

### The Issue

First let’s look at the current behavior when you are dealing with this issue.

![](/images/pkgmgmt1.png)

When running Find-Module, you’ll see some error’s being thrown. Most notably about the missing PowerShellGet module provider. When we check the Package Providers available to us, we can indeed see it is missing. Note that if you are on an older build, PowerShellGet provider was called PSModule so your error could be slightly different.

The source of the issue lays within multiple PackageManagement modules being present on the system.

![](/images/pkgmgmt2.png)

This can occur if you have been on Windows flights since early January (pre RTM) this year. If you installed Windows 10 RTM and are on the Windows 10 flights since Windows 10 RTM, you should not see this issue.

The PackageManagement module used to be under $PSHOME\Modules folder in earlier Windows builds (before Windows 10 RTM) but as part of Windows 10 RTM it is moved to the $env:ProgramFiles\WindowsPowerShell\Modules folder. You might not have noticed this earlier as the module version was the same but with the November update the version was incremented to 1.0.0.1.

Modules are loaded with precedence from $PSHOME\Modules over Program Files modules so the incompatible v1.0.0.0 module is loaded by default.

![](/images/pkgmgmt3.png)

### The fix

There is no easy way to remove files from System32 when they are owned by the TrustedInstaller principal. It requires you to have Security Privileges enabled which you don’t have by default when running PowerShell (not even when elevated). Although there are a lot of cool tricks out there to deal with this situation (e.g. see Boe Prox’s awesome module <http://learn-powershell.net/2015/06/03/managing-privileges-using-poshprivilege>), this goes far beyond what this blog post is intended to accomplish (fixing your PackageManagement problem).

Open your Explorer and navigate to %windir%\System32\WindowsPowerShell\v1.0\Modules. Select the PackageManagement folder and press the delete button on your keyboard. Select Yes and on the UAC prompt select Continue. The folder should now be deleted and your PackageManagement will work again. Note that if files are locked, you probably have the module loaded in one of your PowerShell sessions.

![](/images/pkgmgmt4.png)