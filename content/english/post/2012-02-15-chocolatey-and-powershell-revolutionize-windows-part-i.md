---
title: Chocolatey and PowerShell – Revolutionize Windows, Part I
author: Rob Reynolds
type: post
date: 2012-02-15T19:00:41+00:00
url: /2012/02/15/chocolatey-and-powershell-revolutionize-windows-part-i/
views:
  - 17827
post_views_count:
  - 2300
categories:
  - Package Management
tags:
  - Chocolatey
  - Package Management

---
Ten years ago we did things a certain way. Five years ago we thought the things we did ten years ago were antiquated. Tomorrow you will wonder how you ever used Windows without PowerShell and Chocolatey.

![](/images/choc1.png)

Bold statements? Perhaps. Let’s think about a scenario for a moment. You run an open source project and for someone to hack on your project, they need to have [Ruby][1], Ruby’s [DevKit][2] and [NuGet][3] installed. That’s kind of a large barrier to entry. What if there was a project out there where you could run **one command** and it would completely set up the development environment with all the tools you needed to hack on that project?

Shift for a second. What if there was a place that was like an app store that had free/open source tools and applications for Windows? What if installing these apps/tools was one line of code? What if I could have packages that have dependencies on multiple tools?

### What if this already exists today?

Do I have your attention? Let’s talk about what [Chocolatey][4] (http://chocolatey.org) brings to the table. Chocolatey is an enabler. It enables you to get tools, to subscribe to the idea of global silent application installers, and to enable easier collaboration with others on windows. Chocolatey is a machine package manager that is kind of like [apt][5][&#8211;][5][get][5] with a Windows flavor.

If you are unfamiliar with [apt][6] ([Advanced][5][Package][5][Tool][5]), it is the concept of software, configuration, components, everything (including the Linux kernel in Debian), built into packages that you can install on a Linux machine. Apt handles retrieval, configuration, installation and updating of software packages.

Let’s explore the concepts of Chocolatey and how it can shift your thinking about what’s possible with Windows.

### Tools Enabler

Chocolatey, like [Ruby][7][Gems][7], will allow people to include executables in their packages. Ruby Gems, if you are unfamiliar, are software packages that contain a library or application. The application gems are synonymous with what I will describe as “tools” because they are brought to your system, but they are not “installed” to windows.

If you include an executable in a package or download one to the package folder at runtime, Chocolatey will set up a batch command file on the path that links to that executable so that it can be called from anywhere on the system. Let’s say I want to install something like StatLight, which is a tool for testing Silverlight applications. The NuGet package literally has the executable in it, so all I need to run is (Note: all Chocolatey commands can be run without PowerShell thus why they don’t look like PowerShell verbs):

```powershell
PS \> chocolatey install statlight
```

It picks up the StatLight executable in the package and links to it, so I can run:

```powershell
PS \> statlight
```

The command redirects me to the actual statlight.exe and allows me to use the tool.

![](/images/choc2.png)

### Global Silent Application Installer

At one point or another you may have worked with the concept of keeping a set of installers and MSIs on some company drive with silent install batches so that you could set your machine up. You were stuck to that version. When I started working on chocolatey, I had a couple of years’ experience working on making silent installers at two companies. The biggest roadblocks were keeping all of these applications up to date. Worse still, making sure everyone was on a standard version after you updated the package.

Chocolatey solves both of these problems because it never requires you to download these items ahead of time and it makes updating as easy as installing. To install an application like msysGit I just need to type:

```powershell
PS \> cinst msysgit
```

Press enter and wait for it to finish. When it completes,  msysGit is installed on my machine. It downloads the MSI from the distribution source and invokes a silent install on it.

![](/images/choc3.png)

![](/images/choc4.png)

That&#8217;s part one, in the next segment we&#8217;ll cover updates, setting up a development environment from source and creating packages.

[1]: http://rubyinstaller.org/
[2]: http://rubyinstaller.org/add-ons/devkit/
[3]: http://nuget.org/
[4]: http://chocolatey.org/
[5]: http://en.wikipedia.org/wiki/Advanced_Packaging_Tool
[6]: http://wiki.debian.org/Apt
[7]: http://rubygems.org/