---
title: '#PSTip Selecting/copying the current line in PowerShell ISE'
author: Ravikanth C
type: post
date: 2012-08-20T18:00:46+00:00
url: /2012/08/20/pstip-selectingcopying-the-current-line-in-powershell-ise/
views:
  - 8441
post_views_count:
  - 2057
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Windows PowerShell Integrated Scripting Environment (ISE) is my first preferred script editor. Especially, with all the new cool features introduced in version 3.0. I love working with ISE and try to make my life easier when using the same. Fortunately, ISE exposes the scripting object model &#8212; $psISE. This object model makes it easy to extend ISE functionality and this tip is about one of those aspects.

Very often, I end up selecting and copying the current line from ISE script editor to a console prompt or into a document or email I am working on. If you have looked at ISE menu or keyboard shortcuts closely, there is no keyboard shortcut or ISE menu item to select current line.

This is where $psISE kicks in.

In the ISE scripting object model, the current line is represented by the property called CaretLine. This can be accessed using:

<pre class="brush: powershell; title: ; notranslate" title="">$psISE.CurrentFile.Editor.CaretLine
</pre>

Now, using the same object model, we can select the current line using :

<pre class="brush: powershell; title: ; notranslate" title="">$psISE.CurrentFile.Editor.SelectCaretLine()
</pre>

Now, coming to the fun part of adding this as a menu item &#8212; once again using $psISE scripting object model:

```
$ScriptBlock = {
	$psISE.CurrentFile.Editor.SelectCaretLine()
}

$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Select _Current Line", $ScriptBlock, "Ctrl+Alt+C")
```

This is it. Once you execute the above code snippet and press Ctrl+Alt+C, the current line in the script is selected. Now, extending this to copy the current line isn&#8217;t tough. Let us see that as well.

```
$ScriptBlock = {
	$psISE.CurrentFile.Editor.SelectCaretLine()
 	$psISE.CurrentFile.Editor.SelectedText | clip
}

$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Copy _Current Line", $ScriptBlock, "Alt+C")
```

Simple! Isn&#8217;t it?