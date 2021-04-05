---
title: ISESteroids 2.6 with new version of PSSharper released
author: Tobias Weltner
type: post
date: 2016-08-10T20:54:56+00:00
url: /2016/08/10/isesteroids-2-6-with-new-version-of-pssharper-released/
views:
  - 17738
post_views_count:
  - 2891
categories:
  - News
tags:
  - News

---
PowerShell is everywhere these days. And solid PowerShell skills are probably among the most important personal career factors for an IT professional today. Such skills are crucial for enterprises as well. They need safe and state-of-the-art PowerShell code, and not copy/pasted code fragments that may or may not fit the intended purpose.

The true challenge is that daily business often does not leave the room for education. Most IT professionals learn PowerShell “on the job”, and even experienced PowerShell folks often work under time pressure, occasionally missing best practices and security issues. 

PSSharper V2, shipping in ISESteroids 2.6, aims to solve both needs. It helps you improve your personal skills that you will keep forever, and helps enterprises get better code quality, while working on daily tasks. So there is no extra time needed, and in fact, PSSharper will actually decrease the time needed to write PowerShell solutions. Let’s check out how. 

### Unobtrusive Suggestions

PSSharper works pretty much like your personal PowerShell buddy looking over your shoulder while you are coding. When your code violates a best practice or has other issues, PSSharper adds unobtrusive adornments to your code. Adornments can be squiggles in different colors, or text attributes such as strike-through or ghosting. Have a look:

![](/images/ises1.png)

In the example code, you see different types of squiggles, and the parameter $Path appears ghosted, indicating that it may be undefined. When you hover over an adornment, a tooltip explains the issue to you so you can learn what’s wrong here. Let’s hover over the ghosted parameter:

![](/images/ises2.png)

The tooltip reports two issues: the parameter may be undefined, because it is an optional parameter with no default value, and it has no type constraint, so anything could be assigned to it. With the help of the tooltips, you can quickly screen your code and see what might need further improvements. Let’s hover over the command dir:

![](/images/ises3.png)

Again, there are two issues. The code uses an alias instead of the real command name which can be dangerous because aliases may not always exist, and the command uses positional arguments which introduce both room for errors, and make the code harder to read.

### One-Click Fixes

Most issues can be auto-corrected easily. When you click a squiggle or adornment, the debugger margin displays a light bulb. Click it to see auto-fix options. Let’s first try this with the parameter issue:

![](/images/ises4.png)

For both issues, the context menu offers auto-fixes. Since the parameter has no default value, PSSharper suggests to make it mandatory, and when you choose the fix, the code now looks like this:

![](/images/ises5.png)

Any code change introduced by PSSharper is marked with a light green background so you can review the changes, or undo them, for example, by pressing CTRL+Z. What’s more important: PSSharper not only fixes the issue, it actually shows you what to do. So even if you couldn’t remember how to make a parameter mandatory, by looking at the new code, you immediately see how it is done – and maybe have learned something new.

Let’s fix the command call, too:

![](/images/ises6.png)

The context menu that appears when you click the light bulb again offers fixes for both issues. When you click both fixes, the code now looks like this:

![](/images/ises7.png)

These are just simple examples. In everyday life, you will get many more helpful suggestions. For example, take a look at this awkward command call:

![](/images/ises8.png)

Often, command calls like this stem from Internet code. Simply click the light bulb to fix the issues. The results now looks like this:

![](/images/ises9.png)

PSSharper automatically standardized the call, and removed all ad-hoc shortcuts that were intended for interactive PowerShell commands, not for scripts. The resulting command is much easier to read, helping IT staff to better work together.

Here is another example, before we move on. Can you guess what is wrong with this call?

![](/images/ises10.png)

After fixing the issues, the code looks like this:

![](/images/ises11.png)

One of the detected issues was an absolute path that could prevent your script from running on other machines. PSSharper has automatically replaced part of the absolute path with a suitable Windows environment variable to make your code portable.

### PSSharper Bar

PSSharper evaluates your code in real-time and displays the result in categories at the bottom of the editor window. This gives you a quick overview of all issues in your script:

![](/images/ises12.png)

In the example screen shot, there are 2 warnings, 3 suggestions, and 2 infos. If you do not like this bar, you can hide it by clicking the “cross” icon at the left side. It will then slide out of view. You can always re-open it by either clicking a lightbulb and choosing “Show PSSharper Bar”, or by using the menu “PSSharper”.

To find out more about issues, or even bulk-fix them all, click a category in the PSSharper bar. This opens an add-on panel, listing all issues. Here is an example of a larger script, the legendary “WMI Explorer” from MOW:

![](/images/ises13.png)

The PSSharper bar uses expanders to group the issues, and when you expand a group, you see a quick code preview. The selected item also shows a “magnifying glass” icon. Click it to see more options:

![](/images/ises14.png)

In the context menu, you can fix a single issue, or fix all issues of this kind. Also, at the bottom of the context menu, you can choose “Why is PSSharper suggesting this?” to learn more about the particular issue, and why it is raised.

### Ignoring Rules

A very important aspect in the design of PSSharper was the ability to customize. You decide which rules concern you, and which rules you choose to ignore. For example, while it is best practice to use single quotes for text that has no expandable content, some companies prefer to use double quotes for all strings no matter what. 

To ignore a rule, click the “gear” symbol in the main issue header, and choose “Ignore this rule”. The rule is immediately disabled, all squiggles and adornments related to it are removed, and the rule is moved into the category “Ignored Rule”.

![](/images/ises15.png)

And that’s important: even if you choose to ignore a rule, it will still be evaluated, and you can always see in the PSSharper Bar whether or not there are ignored rules that would apply if they were enabled. So you can easily click “Ignored Rule” in the PSSharper Bar to re-evaluate ignored rules at any time, and re-enable them there if you made up your mind.

### Personalize Squiggles and Adornments

Issues may have different importance to different users, and while some users like to see a big squiggle to get alarmed, others would like a more discreet adornment. This is why you can completely customize the adornments a particular rule uses to alert you.

Simply click the “gear” icon again in a rule header, and choose “Personalize”. You now can pick the squiggle and text adornment used for this rule. You can even change the squiggle colors if you want.

Note that squiggle colors are stored in your current color profile, whereas all other settings are stored in your current PSSharper configuration. We’ll touch that in a second.

### Targeting PowerShell Version

Code analysis is useful only if it produces precise results, and no false alarms. Most obviously you don’t want to get code fix suggestions that turn out to break your code or compatibility. 

This is a challenge because there are so many PowerShell versions out there. Some issues apply only to certain PowerShell versions, and some fixes require PowerShell capabilities not found in all PowerShell versions.

One of the most important settings is therefore to choose the PowerShell version(s) you want to target. This is a trade-off: the more PowerShell versions you (must) target, the less fixes are available, and the less sophisticated will your code get.

To pick the target PowerShell version, in the PSSharper Add-on, click the version link at the top:

![](/images/ises16.png)

As soon as you change the target range of PowerShell versions, PSSharper refreshes its view. Depending on the versions you chose, you will now get a completely different set of issues, fixes, and compatibility information. 

### Compatibility Information

PSSharper integrates what was formerly called “CompatAware”: the PSSharper Bar has a category called “Compat” which lists all issues related to compatibility. With just one quick glance, you can check whether code complies with the PowerShell target versions you set earlier.

![](/images/ises17.png)

Note that this information may not be exhaustive. Just with code issues, we are constantly adding new definitions. PSSharper provides positive lists. All issues listed are indeed something that needs your attention, but there may be more issues that are not (yet) detected by PSSharper.

### Runtime Error Detection

PSSharper treats runtime errors originating in a script just like any other issue. So if you run a script, and a runtime error occurs, the runtime error is listed in the category “Error”. The tooltip displays the original error message (and coincidentally is localized in the below screen shot because it was taken from a German system).

![](/images/ises18.png)

Since runtime errors are just issues to PSSharper, you can auto-fix them. Simply click the light bulb to apply the fix:

![](/images/ises19.png)

Automatically, PSSharper detects the kind of runtime error, and adds the wrapping code for a specific error handler. The resulting code looks like this:

![](/images/ises20.png)

As you see, PSSharper does all the tricks parts. It detects that the particular error was of type Microsoft.PowerShell.Commands.ServiceCommandException, adds a specific catch clause, and even adds the –ErrorAction Stop to the cmdlet, ensuring that the exception can be caught. 

Inside the error handler, the error information is retrieved and written to a custom object that you now can use to log the error, or handle it in other ways.

### Batch Fixing

You can even polish entire scripts, and batch-apply fixers. This however should be done with care because all code changes are on a best effort basis, and it is your risk to apply them. When you apply a single fix, you can immediately double-check the result. When you bulk-apply fixes, this is different. And that’s why “Batch Run” mode is not enabled by default. 

To batch-enable a particular rule, in the PSSharper Add-on click the “gear” icon in an issue group header, and choose “Batch Enabled”. You can then run all batch-enabled rules in bulk by clicking the button “Batch Run” at the bottom of the add-on panel.

![](/images/ises21.png)

### Creating PSSharper Configurations

Maybe in one scenario, you want PSSharper to target only major issues, and in another, you want a complete issue list. Or one day you’d like noticeable squiggles for issues, and on another, you prefer minimal adornments.

PSSharper configuration files help you switch. Simply configure PSSharper the way you like, then click the button with the downwards triangle in the upper right corner of the PSSharper Add-on. It lets you save your current settings, and also offers to load different configurations you may have saved earlier.

### Reporting

You can even export the information collected by PSSharper in object form, then use your own PowerShell code to derive reports or other useful things from it. 

Simply click the button “Export” at the bottom of the PSSharper Add-on. It actually runs the cmdlet Get-PSSharperData which provides the PSSharper information for the currently selected editor pane.

### Next Steps

ISESteroids (and PSSharper) is a work in progress, driven by your excellent feedback and our usability labs results, and backed by the license revenues it generates. As always, even major updates like this one are completely free of charge for anyone with a license – no need for maintenance fees or update licenses.

8 weeks ago, PSSharper V1 surfaced. Thanks to your great feedback, we added all of what you just have seen, like real-time code analysis, and auto-fixes. This is just a milestone, not the destination. Please provide suggestions for additional rules here: http://www.powertheshell.com/isesteroids2-2/support/.

We are working hard to make the internal PSSharper engine compatible with PSScriptAnalyzer rules. And we are aiming to extend compatibility information which currently does not include classes and enums.

### Go For It: Test Drive ISESteroids and PSSharper

It takes less than 2 minutes for you to test drive and play with the examples shown in this article, and ISESteroids runs with all features for a full 10 testing days, so plenty of time to polish and analyze lots of scripts. ISESteroids is a simple PowerShell module and loads into the built-in PowerShell ISE. No special privileges required.

To download and install the module, we strongly recommend that you launch the PowerShell ISE and use the following simple command inside of it:

Install-Module –Name ISESteroids –Scope CurrentUser

Next, load the extension into the PowerShell ISE – and that is really it:

Start-Steroids

If you have installed the module before, either use Update-Module –Name ISESteroids, or use –Force with Install-Module.

If the cmdlets Install-Module and Update-Module are missing, you are probably using PowerShell 4 or PowerShell 3. Either upgrade to PowerShell 5 (visit www.powershellgallery.com  for the download link), or on the same webpage, download and install the PowerShellGet installer package. It adds the two cmdlets used above to your existing PowerShell version.

If you can’t do that either, you can download and install ISESteroids as a ZIP file here: http://www.powertheshell.com/isesteroids2-2/download/

When ISESteroids starts, it displays the location from where it was loaded in the status bar at the bottom of its window. This easily allows you to detect and remove orphaned and older versions of this module in case you have installed it elsewhere before.

&nbsp;

&nbsp;