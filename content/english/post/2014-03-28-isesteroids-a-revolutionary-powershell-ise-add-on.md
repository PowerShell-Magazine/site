---
title: ISESteroids – A Revolutionary PowerShell ISE Add-On
author: Tobias Weltner
type: post
date: 2014-03-28T17:14:58+00:00
url: /2014/03/28/isesteroids-a-revolutionary-powershell-ise-add-on/
categories:
  - Module Spotlight
  - ISESteroids
tags:
  - Modules
  - ISESteroids

---
If you love the built-in ISE editor that ships with PowerShell 3.0 and 4.0, but you were missing some professional editor capabilities, then ISESteroids add them for you. It is a commercial PowerShell module that seamlessly integrates with the ISE editor, written by PowerShell MVP Tobias Weltner (me).

The primary goal of ISESteroids was to keep the beautiful simple UI, and not turn the editor into something as complex as the control panel of a nuclear power plant. So once you start ISESteroids, you will notice only a few changes. Most of the vast power built into ISESteroids shows up once you need it.

Let me take you on a quick walk-through! The version I am showing to you is 1.0.3.30. If you have already downloaded an earlier version (the version information appears when you load ISESteroids), make sure you download the latest version.

![](/images/ISESteroids.png)

### Installation

ISESteroids are no separate application. They are a PowerShell module with simple copy/paste deployment. So you neither need Administrator privileges nor other prerequisites to use it.

Simply get yourself a trial copy here: <a href="http://www.powertheshell.com/isesteroids/download/" title="ISESteroids Download page" target="_blank">http://www.powertheshell.com/isesteroids/download/</a>. It downloads as a .zip file. Unfortunately, Google Chrome is confused by the .zip content and spits out a warning. Feel free to run the .zip through your favorite antivirus scanner if you are concerned.

Next, the single most important step is to unblock the .zip file, or else ISESteroids cannot run. So before you unpack the .zip, right-click it, choose &#8220;Properties&#8221;, then click &#8220;Unblock&#8221;.

Once you unpacked the .zip file, you get a folder called &#8220;ISESteroids&#8221;. Simply copy it to your modules folder. If you do not know where that is, run this line in PowerShell to find out the places where PowerShell looks for modules:

<pre class="brush: powershell; title: ; notranslate" title="">$env:PSModulePath -split ';'
</pre>

### Loading ISESteroids

To launch ISESteroids, start the PowerShell ISE editor, then enter this:

<pre class="brush: powershell; title: ; notranslate" title="">Start-Steroids
</pre>

This will load the extension in a matter of a few seconds, and after you accepted the EULA (which essentially tells you that you are using ISESteroids on your own risk), you will see additional buttons appear in the toolbar. Also, a help window opens.

### Visual Helpers

ISESteroids adds green and blue squiggle lines. They mark code that is OK but does not adhere to best practices. For example, if you use double-quotes with text that does not contain expandable content, it gets a squiggle line. Or, if you use alias names, it again gets squiggled in real-time.

Click on a squiggle to find out about the problem in the ISE status bar. The status bar also provides links that help you fix the problem. So while you code, you learn about best practices and produce more reliable code on the fly.

Another great visual helper is the colorizer that colorizes the current line, so you always know where you are. And if you place your cursor on a brace, the area enclosed by it will be highlighted.

### Instant Help

If the help pane is visible, you get help for almost anything you click in your code. So no need anymore for searching manually. ISESteroids comes with help for most default cmdlets, so even if you did not download the PowerShell help files, help is at your fingertips.

The help is fully interactive. When you hover over examples in the help, they turn blue, and when you click, the example code is entered into the console or editor. The code will never be executed automatically, though. You still have to press ENTER to actually run the code.

When you hold ALT while you click a cmdlet or other part of your script, help opens in a separate window. This is especially useful if you have a secondary screen. The separate help window uses a tab style interface, so you can open multiple help topics in one window.

### Instant Completion

Are you tired of entering always the same control structures? Try this: enter func, then hold SHIFT and add a space. The auto-completion add-on appears and suggests a couple of commonly used function definitions. Simply use your arrow keys to select the one you need, press ENTER, and you are done.

The same works with many other keywords such as if, else, or try. It is not yet complete and will be expanded.

### Auto-Indent

It takes just a click on the auto-indent button (or ALT+I) to automatically indent braces. ISESteroids changes the default indent from 4 to 2, but if you&#8217;d like to adjust this, simply right-click the auto-indent button in the toolbar and choose indent size and indent type. Your changes will be applied the next time you use automatic indent.

### Navigation Bar

ISESteroids adds an optional navigation bar on top of your script. You can show and hide it via ALT+N. It sports a drop-down list on the left with all of the functions in your script for easy navigation. On the right, you get a real-time search box. Anything you enter here will be highlighted in the editor, using multiple selections. When you press ENTER in this search field, ISESteroids take you to the next highlighted instance.

And when you right-click on a function you use elsewhere in your script, the context menu now sports a &#8220;Go To Definition&#8221; that takes you to the function definition.

If the function resides in a separate script or module, it is opened in a separate script pane.

### Smart Selection

ISESteroids supports language-based selections. Simply place your cursor into a PowerShell statement, and press CTRL+Q. The expression the cursor is in will be selected, and the status bar tells you what the expression is. Press CTRL+Q again to select the parent expression. By pressing CTRL+Q multiple times, you can easily select exactly the code you want.

### Block Commenting

To comment code out, select the code, then press CTRL+B. The code is enclosed in a block comment. The same keyboard shortcut uncomments again.

And when you right-click on a comment, you can remove this comment or all comments via new commands in your context menu.

### Variable Monitor

To open the built-in variable monitor, click on the &#8220;eye&#8221; toolbar button. The variable monitor has a blue selection menu at the right side of the text filter box. By default, it shows only user variables, but if you want, you can also view system variables or all variables.

Right-click a variable and choose &#8220;Monitor&#8221; to place it on a custom watch list.

The variable monitor updates after each execution, so you can actually watch your variables change as you go, even during debugging.

When you right-click a variable in the variable monitor, you can enable sophisticated debugger breakpoints such as &#8220;Break on read&#8221;, &#8220;Break on write&#8221;, or &#8220;Break on type change&#8221;. &#8220;Break on type change&#8221; breaks whenever a variable changes data type.

### Debugger Enhancements

ISESteroids adds a new &#8220;bug&#8221; toolbar button that enables you to set and clear breakpoints easily (provided you have saved your script first, because debugging is generally only supported with files).

Once PowerShell hits a breakpoint, additional buttons become visible that enable you to easily step in, step over or out, and to stop the debugging session.

### Script Pane Management

When you have multiple script panes open, right-click on a script pane tab to open a context menu. It allows you to &#8220;Close All Others&#8221;, or discard untitled documents. It also includes commands to send your current script via email as attachment, a .zip file, or protected .zip file, for example if you want to share it with a colleague.

### Fine Tuning

Almost everything can be tailored to your needs. Click the &#8220;blue pill&#8221; toolbar button to open an additional toolbar. It lets you enable and disable a lot of the editing wizards&#8211;for example, you can turn on and off the squiggles, or the quote completion.

Right-clicking a button in this secondary toolbar lets you open help, or change settings. For example, you can freely adjust the selection colors to your taste.

### Code Signing

To digitally sign a script, simply click the sign script button in the toolbar. It looks like a document with a pen. If you have a code-signing certificate installed, it is used. Else, ISESteroids offers to create one for you. Right-click the button to see more signing options.

With the &#8220;spy glass&#8221; toolbar button, you can then check the current script. It will tell you if there is a signature, if it is valid, and who exactly signed the script.

When you right click a digital signature, you can validate the signature or remove it, right from the context menu.

### Decompiling Cmdlets

If you&#8217;d like to know what exactly occurs inside a cmdlet, simply right-click it and choose &#8220;Source Code&#8221;-> &#8220;Decompile&#8221;. The free ILSpy decompiler opens and presents you with the cmdlet source code. This works with raw .NET types as well.

The same menu also allows you to derive proxy functions from cmdlets or turn a cmdlet into a PowerShell function, so you can better understand how you would define parameters in your own functions.

### Creating Functions from Selection

Typically, when you play with PowerShell, you will eventually end up with code similar to this:

```
$text = 'Hello World!'
$rate = 0

$obj = New-Object -ComObject Sapi.SpVoice
$obj.Rate = $rate
$null = $obj.Speak($text)
```

This would for example speak out the text in $text, at least if you enabled your speakers.

To turn ad-hoc code like this into a PowerShell function, simply select the entire code, then right-click the selection and choose &#8220;Turn into function&#8221;. A wizard opens.

Please note: prior to ISESteroids version 1.0.3.30, the wizard had a bug that could cause an exception, so make sure you are using at least this version.

First choose a function name in the upper part: a drop-down box offers all of the approved PowerShell verbs.

Next, look at the list of variables that the wizard would turn into function parameters. Deselect &#8220;-obj&#8221; because you want to keep this variable private. Once you click OK, the function is generated and will look like this:

```
function Out-Voice
{
<#
    .Synopsis
    	Short Description
    .DESCRIPTION
    	Detailed Description
    .EXAMPLE
        Out-Voice
        explains how to use the command
        can be multiple lines
    .EXAMPLE
        Out-Voice
        another example
        can have as many examples as you like
#>

	[CmdletBinding()]
	param
	(
        [Parameter(Mandatory=$false,Position=0)]
        [System.String]
        $text = 'Hello World!',

        [Parameter(Mandatory=$false,Position=1)]
        [System.Int32]
        $rate = 0
     )

    $obj = New-Object -ComObject Sapi.SpVoice
    $obj.Rate = $rate
    $null = $obj.Speak($text)
}
```

Take a minute to fill out the help block so your new function gets PowerShell help.

### Turning Functions into Modules

Once you have created a function, you can easily move it into your very own PowerShell module. Simply right-click the &#8220;function&#8221; keyword, and choose &#8220;Export To Module&#8221;.

A wizard opens. You can either enter the name of a brand new module you want to create, or&#8211;if you have created modules with this wizard before&#8211;you can choose an existing module to host your function.

That&#8217;s it. The module is created, and you can use your new command in all PowerShell shells, without restart or loading anything.

There is a lot more in ISESteroids, but this quick walk-through hopefully has given you a good starting point for your own test drive.

Remember that ISESteroids is a commercial project. It will run for free for 10 distinct testing days. After that, you need a license. License revenues will fund this project, so if you like what you see, the ISESteroids makers greatly appreciate if you do your share and provide some revenue.

License lasts forever for the version you purchased. There is an introductory offer going on right now that includes 1-year free support and 1-year free updates.