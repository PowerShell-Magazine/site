---
title: Building Advanced User Interfaces with ShowUI
author: Joel "Jaykul" Bennett
type: post
date: 2012-04-25T18:00:41+00:00
url: /2012/04/25/building-advanced-user-interfaces-with-showui/
views:
  - 34310
post_views_count:
  - 3275
categories:
  - Module Spotlight
  - ShowUI
tags:
  - Modules
  - ShowUI

---
I’ve blogged a lot of quick demos with [ShowUI][1] (and its predecessors) in the last few years, but as things ramp up with ShowUI and it starts getting more attention at the enterprise level, a lot of the questions that we&#8217;ve been getting on the [ShowUI forums][2] focus on the professional touches that make a user interface feel like an application, and the challenges of pulling together different controls into a single user interface in a reusable way.

In this article we’ll walk through the process of building a reusable solution with ShowUI, demonstrating how to create re-useable control functions and how to pull them together into a single interface.  The challenge for today will be a script to manage displaying a collection of data… with a rich interface for sorting and filtering.

In the interests of showing off some of the data-binding and templating features of WPF 3.5 and the built-in capabilities of PowerShell and ShowUI, we’re going to use a [ListView][3] with a [GridView][4] rather than a [DataGrid][5] (which is available in .Net 4, or in the [WPFToolkit][6] for .Net 3.5).

To get started with our display we need a ListView with a GridView which can sort and filter. We’ll create a reusable control for this, since it’s obviously something we’ll have a lot of use for.  There’s been some guidance from the ShowUI team on how to create these reusable controls, but here are the basics:

  1. Create a function which outputs a WPF control (generally, a grid or panel with any other controls inside, but that’s not important).
  2. Include all of the common ShowUI parameters (Name, Row, Column, RowSpan, ColumnSpan, Width, Height, Top, Left, Dock, Show and AsJob).
  3. On the top-level control that you output, use the –ControlName parameter to create a scope for your control.
  4. On the top-level control that you output, pass through the common ShowUI parameters.

As a sidebar comment: If you’re new to ShowUI (or WPF), you need to know a couple of additional points:

  * Most of the built-in functions with the “New” verb in ShowUI are essentially wrappers for working with a specific control (or class) in WPF, and the parameters map directly to properties and events on those controls. As a result, frequently the best documentation for a ShowUI command is the MSDN documentation for the control. A case in point is the [ListView][7] control that we’re working with, where the MSDN documentation includes an example of using it with a [GridView][8] and information about [virtualizing and other optimizations][9] that can be performed (and what would disable them).
  * In ShowUI scripts, I usually use aliases like “ListView” instead of New-ListView. The reason for this is that the aliases are bound with the module name, so “ListView” is actually: ShowUI\New-ListView.  Because of that, they’re actually less likely to conflict with other modules than the name by itself (specifically, PowerTab has a New-TabItem command which has occasionally caused problems, but using the “TabItem” eliminates the problem).

[Listing 1][10] shows the code for Show-GridView.  As you can see, I’ve written a control which accepts one or more InputObjects (including from the pipeline) and displays them in a GridView, given a list of properties, and outputs the selected items when the window is closed. It also supports sorting the GridView when a column header is clicked.  The code for that sorting is somewhat complex, but it’s just a translation into PowerShell of the MSDN article [How to: Sort a GridView Column When a Header Is Clicked][11], so I’m not going to explain again it here, that article is a good resource, and the logic applies to all [CollectionView][12] controls. Note that Show-GridView requires ShowUI 1.3 or later because of the syntax of the Add-EventHandler call, which doesn’t handle bubble-up events in ShowUI 1.1.

Already we can use our control and pipe the output from it to additional commands. Anything selected in the GridView will be output when the window is closed (but if nothing is selected, everything will be output, so be careful, there’s no “cancel”). However, it doesn’t sort yet, and it also doesn’t have a way to filter the controls. Here’s an example:

```powershell
PS> Get-ChildItem | Show-GridView -Property Mode, Length,

LastWriteTime, Name –Show -Name (split-path $Pwd -leaf)
```


![](/images/ShowUI_Lisiting1.png)

You do have to specify the properties that you want displayed, because we haven’t included anything to get them from PowerShell format files or through reflection, and you have to be careful: although PowerShell isn’t case sensitive, WPF is – so the data-binding which happens with the columns is case sensitive.  This means that you have to put (for example) “LastWriteTime” rather than “lastwritetime” or you won’t get any values in the column.

To handle sorting we need to use Add-EventHandler, because the event we want to use to change the sorting isn’t an event of the ListView, it’s the Click event of the [GridViewColumnHeader][4](s) which get added to the ListView automatically.  To call Add-EventHandler we can just add code to the top of the On_Loaded event handler we already have … and we can handle the sort one of two ways:

[Listing 2][10] shows a way to perform the sort using the native WPF sorting features.  This method is by far the fastest and smoothest. The problem with it is that in PowerShell, we sometimes display data that’s not all the same. For instance, when we displayed Get-ChildItem above, we’ll probably have a mix of files and folders, which will mean that the “Length” column will be empty sometimes, which will break sorting by that column (try it yourself, you’ll see the errors in the console and the sort won’t work).

[Listing 3][10] shows an alternate way to perform the sort. Rather than using WPF’s sorting mechanism, we can sort the original items in PowerShell, and then (re)assign the sorted collection to the view.  This method is generally slower, but it’s guaranteed to work on anything that PowerShell outputs, because PowerShell’s sorting has better handling for missing values.

To handle filtering, we want to add a [TextBox][13] where you can type text, and then filter the list based on that text. Once again, we can filter in two ways.  Hopefully you know how to use Where-Object to filter in PowerShell, so I’m going to skip over that and leave it as an exercise for you. One warning: when filtering, you’ll need to store a copy of the original list in the “Resources” of the control, because otherwise each filter will be permanent and cumulative!

I’ve written [Listing 4][10] as a generic control which takes any [ItemsControl][14] (or derived control) which has an Items or ItemsSource property and a [CollectionView][12], and can filter it (so it will work with our Show-GridView, but also with any similar control). Since I don’t know the name of the ItemsControl, there’s a lot of extra code in both the Filter ScriptBlock (starting on line 42), and the Border Loaded event handler (starting on line 90), which serves primarily to find the control and its CollectionView. It’s somewhat convoluted, but it makes the Show-FilteredItemsControl more reuseable.

With the code from Listing 1 (with either Listing 2 or 3 in its “On_Loaded” handler) and Listing 4 together, we can now create sortable, filterable lists like this:

```powershell
PS> Show-FilteredItemsControl {

Get-ChildItem | Show-GridView -Property Mode, Length,
LastWriteTime, Name
} –Show
```


And use them to stop processes:

<pre>PS&gt; Show-FilteredItemsControl {
&gt;&gt; Get-Process | Show-GridView -Property Handles, VM, CPU,
&gt;&gt; Id, ProcessName
&gt;&gt; } –Show | Stop-Process
&gt;&gt;</pre>

![](/images/ShowUI2.png)

It’s a bit complicated, but it is possible to automatically determine which columns should be used (based on the PowerShell formatting rules), and to provide the ability to specify custom labels for the columns. Additionally, we could add UI for creating new items into the collection and removing items, so that this code could be used for editing lists of any sort of objects … but I’ll leave that as future topic.

As always, if you have questions, feel free to ask them on [the ShowUI forums][2].  [Download][10] the scripts for this article.

[1]: http://show-ui.com/
[2]: http://showui.codeplex.com/discussions
[3]: http://msdn.microsoft.com/en-us/library/system.windows.controls.listview
[4]: http://msdn.microsoft.com/en-us/library/system.windows.controls.gridviewcolumnheader
[5]: http://msdn.microsoft.com/en-us/library/system.windows.controls.datagrid
[6]: http://wpf.codeplex.com/releases/view/40535
[7]: http://msdn.microsoft.com/en-us/library/System.Windows.Controls.ListView
[8]: http://msdn.microsoft.com/en-us/library/system.windows.controls.gridview
[9]: http://msdn.microsoft.com/en-us/library/cc716879
[10]: /images/ShowUI.zip
[11]: http://msdn.microsoft.com/en-us/library/ms745786.aspx
[12]: http://msdn.microsoft.com/en-us/library/system.windows.data.collectionview
[13]: http://msdn.microsoft.com/en-us/library/system.windows.controls.textbox
[14]: http://msdn.microsoft.com/en-us/library/system.windows.controls.itemscontrol