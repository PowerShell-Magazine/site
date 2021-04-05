---
title: 'PowerShell v3 ISE and ISE scripting model changes & improvements'
author: Ravikanth C
type: post
date: 2011-09-28T13:29:27+00:00
url: /2011/09/28/powershell-v3-ise-and-ise-scripting-model-changes-improvements/
views:
  - 31531
post_views_count:
  - 4007
categories:
  - PowerShell ISE
  - News
tags:
  - News
  - PowerShell ISE

---
PowerShell Integrated Scripting Environment (ISE) in version 2 was a great addition for people who wanted a better script editor with syntax highlighting. It provided options to change preferences &#8212; the look and feel &#8212; using a simple scriptable object model. This scripting model was exposed through `$psise` object. It was a great start but nowhere near any of the 3rd-party script editors in the market.

PowerShell ISE in version 3 is a leap ahead of its predecessor. PowerShell ISE in v3 improved the overall functionality by providing:

  * Much improved IntelliSense with drop-down selection for object properties, methods, cmdlet and function names, cmdlet parameters, and argument values.
  * The ability to add additional panes (horizontal or vertical) for custom add-ons
  * Support for XML file highlighting
  * Script regions folding and unfolding
  * Code snippets support
  * Copy with syntax color; when you copy script code from ISE into a rich text editor, the color scheme will be retained.
  * A nice built-in and searchable help (F1) WPF window; it doesn't open a CHM file anymore.
  * Last but not least, a powerful `$psise` object model for extending ISE!

In this article, I will provide a quick overview of what changed under `$psise.Options` (this object contains all properties that define the look and feel of ISE) and then provide detailed information for some of the new properties under `$psise.Options`. The following table (a cheat sheet) shows these changes at a high level.

![](/images/ise1.png)


  <p>
    Now, let us look at some of the new options that can be used to change the ISE&#8217;s functional behavior.
  </p>
### AutoSaveMinuteInterval

This option is new in PowerShell ISE v3 and can be used to set the interval for auto save of scripts. This feature does not auto save the file but it enables auto recovery of unsaved file in case of an ISE crash or unexpected errors. This property accepts an Int16 value which means anything from -32768 to 32767 is a valid value. The following setting will reduce the auto save interval to one minute.

```powershell
$psISE.Options.AutoSaveMinuteInterval = 1
```

Setting a negative value will disable ISE auto save and the ability to recover unsaved files in the event of a crash.

### Most Recently Used (MRU) List & MruCount

In PowerShell ISE v3, there is a most recently used scripts list. By default, this value is set to 10. You can change this by updating `$psise.Options.MruCount` property.

```powershell
$psISE.Options.MruCount = 2
```

Valid values for this property are anything in the range of 0 to 32. A change to this property persists over restart of ISE.

### IntelliSense in Command pane and  Script pane

![](/images/ise2.png)

One of the most important additions to the script editor in version 3 is the IntelliSense drop-down support similar to what you find in Visual Studio.

This feature shows both object properties and methods, cmdlet and function names and parameters in the drop-down. Also, as you hover the property or cmdlet names, you see a tooltip with the type of parameters and parametersets are available.

For properties and methods, you see a list of parameter types, all variants of methods along with possible arguments types and variations.

The new `$psise.Options`object has 5 properties to alter the behavior of this new IntelliSense feature. By default, IntelliSense is enabled in both Script pane and Command pane.

### ShowIntellisenseInCommandPane

This property can be used to disable or enable IntelliSense in Command pane. For example,

```powershell
$psISE.Options.ShowIntellisenseInCommandPane = $false
```

... will disable the IntelliSense drop-down feature in the Command pane.

### ShowIntellisenseInScriptPane

This property under $psise.Options can be used to disable or enable IntelliSense drop-down feature in the Script pane. For example,

```powershell
$psISE.Options.ShowIntellisenseInScriptPane = $false
```

... will disable IntelliSense drop-down in the Script pane.

### IntellisenseTimeOutInSeconds

This property can be used to configure how long ISE should search for an object's properties and methods or cmdlet names or parameters before timing out. By default, this is set to 3 seconds. For example, if you have a huge list of modules, there is a good chance that three seconds may not be enough to get that complete list and start a filter based on what you are typing. This is where you may find increasing the time out value useful. You can do so by changing:

```powershell
$psISE.Options.IntellisenseTimeoutInSeconds = 5
```

If you notice, this property takes signed int32 values. This means, you can set a negative value as well. But, by doing so, ISE will crash the very next moment you do a TAB or put a dot next to a object. This is a bug!

### UseEnterToSelectInCommandPaneIntellisense & UseEnterToSelectInScriptPaneIntellisense

These two properties can be used to configure how the IntelliSense drop-down treats a return keystroke.

By default, UseEnterToSelectInCommandPaneIntellisense is set to FALSE which means navigating to an item in the drop-down and pressing Enter won't select the item. Instead, you either select the item using mouse or use tab to complete the property name. This can be enabled using:

```powershell
$psISE.Options.UseEnterToSelectInCommandPaneIntellisense = $true
```

UseEnterToSelectInScriptPane is set to $true, by default. So, unless you really want to disable that behavior, you don't have to touch this property.

Make a note any changes to these options will persist over ISE restart.

### Zoom

![](/images/ise3.png)

In PowerShell ISE v2, at the bottom right-hand corner, you will see a slider to adjust the size of script font in the Script and all other panes. This isn't really zoom! It just changes the font size as you adjust the slider. However, in PowerShell ISE v3, there is a new slider that really acts like a zoom slider you see in Word, PowerPoint, or Excel.

![](/images/ise4.png)

There is also a new property under `$psise.Options` called <em>Zoom</em>. So, as you move the slider right or left, this property gets updated and not the `FontSize` property. You can also set this property by:

```powershell
$psISE.Options.Zoom = 200
```

The valid values for this property are 20 to 400. I don't really see a practical reason why you'd ever use this in a script. You can always use Ctrl++ and Ctrl+- to update the zoom factor. And, this change persists even after you restart ISE.

### ShowCommandsOnStartup

By default, ISE v3 opens a new vertical pane &#8212; essentially the same as `Show-Command` cmdlet &#8212; called [Commands](http://www.jonathanmedd.net/2011/09/powershell-v3-ise-commands-add-on.html) that helps you access the parameter and help information for all cmdlets available. This add-on can take a good amount of your ISE screen real estate even when you don&#8217;t need it and it opens every time you open ISE. This behavior can be disabled by setting `$psISE.Options.ShowCommandsOnStartup` to $false.

```powershell
$psISE.Options.ShowCommandsOnStartup = $false
```

### ShowOutlining and ShowLineNumbers

In the Script pane of ISE v3, code folding is enabled and like all other 3rd-party commercial/free editors, you can have code regions and fold them. Here is how it looks:

![](/images/ise5.png)

The vertical line next to line numbers is the code outline and that can be disabled by setting `$psise.Options.ShowOutlining` to `$false`. This is how the editor window looks after we disable outlining.

![](/images/ise6.png)

The line numbers in the Script pane aren't new to ISE v3 but the option to disable them is. If you don&#8217;t like to see the line numbers along with the script code, you can disable it by setting `$psise.Options.ShowLineNumbers` to `$false`. Also, make a note that the `#region` and `#endregion` are case sensitive.

### ShowDefaultSnippets

![](/images/ise7.png)

PowerShell ISE v3 supports code snippets like its 3rd-party counterparts. You can press Ctrl+J to bring up a tiny drop-down menu and select a code snippet from that list.

This can be disabled by setting:

```powershell
$psISE.Options.ShowDefaultSnippets = $false
```

### XmlTokenColors

PowerShell ISE v3 supports XML syntax highlighting and the XML token colors can be changed by updating `$psise.Options.XmlTokenColors` hash.

All changes to `$psise.Options` persist over ISE restart and hence we don&#8217;t really need the old trick of using ISE profile to reconfigure the properties every time we open ISE. Make a note that for the changes to persist, ISE has to exit in a graceful manner. In case of a crash, the new changes will not be saved. And, if you want to reset all these properties to their defaults, you can use `$psISE.Options.RestoreDefaults()` method. This resets all `$psise.Options` to what is available in `$psise.Options.DefaultOptions.`

In summary, PowerShell ISE in version 3 brings in lot of goodness. Given the extensible aspects of PowerShell ISE object model, there will be lot of additional features added by the community. In this article, we just saw only a subset of changes to PowerShell Integrated Scripting Environment in version 3. And, we looked only at the `$psise.Options` object and what has changed underneath this object alone. There is more!
