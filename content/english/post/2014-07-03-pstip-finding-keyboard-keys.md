---
title: '#PSTip Finding keyboard keys'
author: Jan Egil Ring
type: post
date: 2014-07-03T18:00:02+00:00
url: /2014/07/03/pstip-finding-keyboard-keys/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
A [previous article][1] in PowerShell Magazine describes how to find keyboard shortcuts in the PowerShell ISE. In this tip we will look at how we can find a key we do not know. The Go To Match keyboard shortcut in the ISE is Ctrl + Oem6. In my case I&#8217;m using a Dell laptop, and I have no clue where the Oem6 keyboard key is.

As a workaround, we can assign Go To Match a custom key combination in a PowerShell ISE Add-on menu item:

<pre class="brush: powershell; title: ; notranslate" title="">$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('GoToMatch',{$psise.CurrentFile.Editor.GoToMatch()},'Alt+X')
</pre>

But if we really want to locate a keyboard key, we can use the [Console.ReadKey][2] method and simply start pressing buttons:

<pre class="brush: powershell; title: ; notranslate" title="">while($true){[Console]::ReadKey($true)}
</pre>

![](/images/readkey.png)

In my case, Oem6 turned out to be the Norwegian letter å. To verify I tried Ctrl + å in the PowerShell ISE to find a matching bracket, which did work as expected.

[1]: /2013/01/29/the-complete-list-of-powershell-ise-3-0-keyboard-shortcuts/
[2]: http://msdn.microsoft.com/en-us/library/system.console.readkey(v=vs.110).aspx