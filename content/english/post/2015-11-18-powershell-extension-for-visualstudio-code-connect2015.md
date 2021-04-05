---
title: 'PowerShell Extension for #VisualStudio Code #Connect2015'
author: Ravikanth C
type: post
date: 2015-11-18T16:11:31+00:00
url: /2015/11/18/powershell-extension-for-visualstudio-code-connect2015/
views:
  - 8896
post_views_count:
  - 1246
categories:
  - News
  - VS Code
tags:
  - News
  - VS Code

---
If you are not watching [#VisualStudio #Connect2015][1], you are certainly missing a lot. Because &#8230; @ScottGu just announced a lot of new features including extension support in Visual Studio Code. And. hey, there is a PowerShell extension.

![](/images/vscodeps1.png)

To get this PowerShell extension for VS Code, make sure you first download the latest update. VS Code has to be at version 0.10.1. You will need PowerShell 5.o too on the system where you are installing this.

Once you have the update, open Command Pallete (View Menu) and type _&#8216;ext install PowerShell&#8217; _and click on the extension to install it.

![](/images/vscodeps2.png)

This is it. Once you are done, don&#8217;t forget to check the [readme][2]. Check out this [PowerShell product team announcement][3] too. This extension is open sourced and [available on Github][4].

If you are developer interested in enabling PowerShell support in an editor, do check out the Github repo that has the [PowerShell Editor Services][5] code. PowerShell Editor Services provides common functionality that is needed to enable a consistent and robust PowerShell development experience across multiple editors.

If you are already wondering about PowerShell ISE, read this last line from PowerShell team announcement.

![](/images/vscodeps3.png)

[1]: https://channel9.msdn.com/
[2]: https://raw.githubusercontent.com/PowerShell/vscode-powershell/master/examples/README.md
[3]: http://blogs.msdn.com/b/powershell/archive/2015/11/17/announcing-windows-powershell-for-visual-studio-code-and-more.aspx
[4]: https://github.com/PowerShell/vscode-powershell
[5]: https://github.com/PowerShell/PowerShellEditorServices