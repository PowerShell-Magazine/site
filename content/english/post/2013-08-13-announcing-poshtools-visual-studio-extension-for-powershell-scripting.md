---
title: Announcing PoshTools – Visual Studio extension for PowerShell Scripting
author: Ravikanth C
type: post
date: 2013-08-13T15:49:17+00:00
url: /2013/08/13/announcing-poshtools-visual-studio-extension-for-powershell-scripting/
categories:
  - News
tags:
  - News

---
Ever wondered why isn’t there support for writing PowerShell scripts in Visual Studio? I have had several projects where I needed to integrate PowerShell scripts into a C# solution I created in Visual Studio. For this, I needed to use multiple editors – one for C# and another for PowerShell scripts. I am sure that I am not the only one looking for such integration between Visual Studio and PowerShell.

[Adam Driscoll][1], a fellow PowerShell MVP, [announced][2] that he is working on [PoshTools][3] &#8211; a reincarnation of [PowerGUIVSX][4]. PoshTools is a Visual Studio extension to enable writing PowerShell scripts and supports full IntelliSense capabilities for PowerShell inside the Visual Studio Editor. This new integration has no dependency on PowerGUI editor.

This project is hosted on [GitHub][3]. Some of the features included in the initial release are:

  * Syntax Highlighting
  * IntelliSense
  * Debugging
  * Breakpoints (some known issues)
  * Breakpoint window
  * Locals window
  * Call Stack window
  * Output Window Support
  * PowerShell Project Support
  * New item templates for PS1, PSD1, and PSM1 files

Go head and explore! We will write a detailed review soon!

[1]: http://csharpening.net/
[2]: http://csharpening.net/?p=1673
[3]: https://github.com/adamdriscoll/poshtools
[4]: http://powerguivsx.codeplex.com/