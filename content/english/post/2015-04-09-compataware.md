---
title: 'CompatAware: Scripting against different PowerShell versions'
author: Tobias Weltner
type: post
date: 2015-04-09T15:45:32+00:00
url: /2015/04/09/compataware/
views:
  - 10213
post_views_count:
  - 1607
categories:
  - How To
tags:
  - How To

---
You may remember &#8220;DLL Hell&#8221; from the old days. A very similar beast lives in our PowerShell ecosystem, too.

Take 5 different PowerShell versions, all with different supported cmdlets, language elements, and parameters, and blend this with hundreds of available modules &#8211; out comes a truly high-proof cocktail, and it becomes evident that it&#8217;s far from trivial to safely develop code for a given platform.

That&#8217;s simply life and the course of technical evolution, but there are ways to make this cocktail much more tolerable. ISESteroids has just added some safety gear for professional PowerShellers called CompatAware.

It is part of a series of assistance systems design to write better code in less time, and improving your PowerShell skills and knowledge while you code. After all, cars today have airbags and navigation systems, too.

Much of what is going to be presented here was inspired by the overwhelmingly positive and valuable community feedback. Thank you much for your trust.

### Targeting a PowerShell Version

Let&#8217;s play a bit with CompatAware and see how it can clean up the version jungle. There is really just one initial setting to care about: specify the PowerShell version you are targeting.

Simply open the (new) menu &#8220;Level&#8221;, and choose &#8220;CompatAware&#8221;, then pick your PowerShell target version. Let&#8217;s assume I need to run a script on some older machines with only PowerShell 2.0 available.

![](/images/compat1.png)

The moment you set a PowerShell target version, ISESteroids starts to highlight all cmdlets, parameters, and language constructs that are not compatible with this version. A yellow squiggle line indicates incompatibilities in real-time, while you code.

That&#8217;s important: you don&#8217;t want to invest hours of work, just to discover at the end that your approach won&#8217;t work at the customer&#8217;s site. Real-time information is what let&#8217;s you rapidly rethink the route you are taking.

In the screenshot, you actually see two squiggles: the blue squiggle indicates a best practice violation (&#8220;do not use aliases in scripts&#8221;), and the yellow squiggle reveals that the command &#8220;dir&#8221; (aka Get-ChildItem) had no -Directory parameter in PowerShell 2.0. It was added only in PowerShell 3.0.

Note that the -Path parameter is not underlined, and neither is the command itself. They both work fine with PowerShell 2.0, and CompatAware is very specific.

![](/images/compat2.png)

Before we dig into the actual compatibility informations delivered by CompatAware, let&#8217;s first take a quick tour of technologies that CompatAware shares with the other assistance systems in ISESteroids.

### Squiggles provide Hover Information

Squiggles serve just one purpose: to catch your attention and curiosity, yet without distracting you too much. Once you&#8217;re hungry for more, simply hover over a squiggle.

Tooltips provide you with extra information, tossing in little doses of PowerShell wisdom when you are ready for it. This way you can refine your scripts and gather extra knowledge while you work.

Squiggles are just the visible tip of an information iceberg and backed by a fully adjustable information system.

Take the descriptive text for example that is provided by yellow squiggle tooltips. It is fully expandable. If you&#8217;d like to add your own compatibility notes, take a peek into the folder &#8220;Compatibility&#8221; inside the ISESteroids program folder. Here, you&#8217;ll find simple text files for all compatibility warnings, and you can edit and expand the descriptive text as you like.

Likewise, when you hover over a blue or green squiggle line, you get helpful information about why your code has just violated best practices. And if ISESteroids can fix the issue for you, a &#8220;light bulb&#8221; shows in the debugger margin, inviting you to click on the bulb to auto-correct all instances of this problem &#8211; and then move on to your next task.

![](/images/compat3.png)

### Intelligent Syntax Error Analysis

Admittedly, some syntax errors are rather obvious, but others can drive you nuts. So ISESteroids enhances the default tooltip messages with helpful hints, too, that help you solve the puzzle fast.

Let&#8217;s take a look at a real-world example: some piece of PowerShell code suddenly starts to display multiple red squiggles, as if struck by measles. What&#8217;s wrong? And where to start?

First of all, tooltips tell you when you are looking at the _wrong_ spot. Some syntax errors are just artefacts of the true underlying error (and please excuse the original German error messages in the screen shots, no time for MUI fiddles right now).

So in the below screenshot, the tooltip indicates that this syntax error is _not_ the root cause, and that you should instead spend your precious time looking at line 7.

![](/images/compat4.png)

When you do this and look at line 7, the tooltip here reveals what needs to be done to get rid of all the red lines: simply &#8220;add a comma&#8221;! Sometimes, the remedy can be really simple once you know what&#8217;s going on. If you have the time to learn more, below this message you get short background infos.

![](/images/compat5.png)

Once you add a comma, all the other syntax errors vanish, and you are good to go.

Of course, ISESteroids cannot know all recipes against all possible syntax errors. Instead, ISESteroids invites you to add your own notes and pieces of wisdom: You can add information to specific syntax errors.

### Attach Wisdom to Syntax Errors

When there is no predefined wisdom attached to a syntax error, the tooltip encourages you to add your own content.

![](/images/compat6.png)

To do this, right-click the warning sign in the debugger margin. This opens a context menu. Simply choose &#8220;Create/Edit Annotations&#8221;. This in turn opens a dialog.

In this dialog, you can now add two things: in the top text box, add a short and precise description of what should be done to get rid of this error. In the text box below, feel free to add explaining descriptions.

Once you are done, click &#8220;Apply&#8221; to save the annotation. Then, click the link &#8220;Open annotation folder&#8221;. This opens the folder where ISESteroids saves all of your annotations &#8211; they are just plain text files. What&#8217;s more interesting is the file name.

![](/images/compat7.png)

In this example, the file name will be &#8220;RCurlynullWhile.txt&#8221;, and the file is stored in the folder _MissingOpenParenthesisAfterKeyword_. As you may have guessed, the folder specifies the kind of syntax error. The file name determines which solution should be displayed.

&#8220;RCurlynullWhile.txt&#8221; really means: the token before the error should be a &#8220;RCurly&#8221;, the token at the error should be &#8220;null&#8221;, and the following token should be &#8220;While&#8221;. So your annotation is highly specific for a certain syntax error scenario. If you want your annotation to be less specific, simply replace each token that you do not care about with a hyphen. So when you rename the file to &#8220;&#8211;While.txt&#8221;, then your annotation will be displayed for all syntax errors of this kind that deal with the &#8220;While&#8221; keyword.

### Dealing With Runtime Errors

Syntax errors are grammatical errors, much like misspelled words in Word. Runtime errors, in contrast, are revealed only when a script runs. The code is fine, but your script is unable to perform what it wanted to do. Let&#8217;s say you were trying to stop a service but your script did not have appropriate privileges.

ISESteroids can detect runtime errors, and will display the same red squiggle line. It even brings runtime errors into view once a script is done, and when you hover over the squiggle, you get information about the runtime error.

![](/images/compat8.png)

Since runtime errors typically require a bit more information, ISESteroids employs a feature called &#8220;MagneticNotes&#8221;. You can see MagneticNotes in action when you look at the screen shot: in an add-on panel, MagneticNotes explain why this runtime error occured, and what needs to be done.

To see MagneticNotes, open the MagneticNotes add-on. You toggle this add-on with the Add-on button displaying a lightning bolt.

![](/images/compat9.png)

Next, click anything inside your editor: a command, a keyword, or a runtime error.

### MagneticNotes &#8211; Attach Information to Tokens and Errors

MagneticNotes are a quick way of attaching arbitrary information to a specific PowerShell token or runtime error. Instead of searching for information, ISESteroids displays the information when you need it. Once you click a token or runtime error, MagneticNotes show the notes you saved earlier. If there are no notes yet attached to the clicked item, you can add notes and save them.

Likewise, to edit existing notes, simply right-click the annotation, and choose &#8220;Edit&#8221;.

MagneticNotes can contain plain text, and code samples. To add code samples, select some code in your editor, copy it to the clipboard, and paste it into the MagneticNotes editor.

![](/images/compat10.png)

### Finding Dependencies

By now you have a solid understanding of how CompatAware is just another assistance system, sharing the same visual concept of squiggles and just-in-time tooltips and MagneticNotes.

Let&#8217;s now return to the original CompatAware feature, and have a final look at the scope of compatibility information it can provide. CompatAware is not limited to PowerShell compatibility issues. It can also pinpoint vital code dependencies, like required external modules or commands.

To get dependency information, open the &#8220;Level&#8221; menu, choose &#8220;CompatAware&#8221;, then enable &#8220;Mark non-default commands&#8221;. Once you do this, ISESteroids adds a yellow squiggle to all commands that are not shipping with PowerShell. This helps you quickly identify commands that originate in optional 3rd party modules.

![](/images/compat11.png)

### Auto-Generating #requires

To play safe, a script should either run successfully, or not run at all. The worst scenario always is a script that stalls in the middle of something, leaving you with a lot of cleanup work.

PowerShell scripts can add a _#requires_ statement, listing all the requirements your script needs to successfully run. Since CompatAware already knows what your script needs, it can derive and auto-generate the correct _#requires_ statement for your script. Existing _#requires_ statements are properly updated.

Simply visit the CompatAware menu, and choose &#8220;Add/Update #requires statement&#8221;, or press CTRL+ALT+R.

You can also use the cmdlet _Update-SteroidsRequiresStatement,_ or run the refactoring addon (the button shows a green checkmark). It will add or update _#requires_ statements automatically as part of the code optimization rules tied to it.

![](/images/compat12.png)

### Auto-Generating Compatibility Reports

CompatAware also shares the compatibility info with you. So you can create compatibility reports, or use the compatibility information in your own reporting tools.

To get a compatibility report, visit again the CompatAware menu, and choose &#8220;Compatibility Report&#8221;. The rich object data can also be directly retrieved:

Get-SteroidsCompatibilityReport -TargetVersion 2 -IncludeDependencies

(Replace &#8220;2&#8221; with the PowerShell version you want to check against).

### What&#8217;s Next?

We have so many more ideas for you, and get nourished by your constant stream of feedback and ideas. One of the features you suggested was a better way to switch between multiple editor panes. Try and hold the right mouse button, then move your mouse wheel. Or hold CTRL, then press PageUp or PageDown.

We now focus on getting ISESteroids 2.0 Final out of the door and would like to celebrate this milestone at the 3rd German PowerShell Conference April 22 in the Philharmony in Essen.

Speaking of which: if you are located in Europe, have at least some _basic_ German language skills yet some _great_ PowerShell passion, then **be there**: April 22+23, Essen, Germany, 12 international speakers including Bruce Payette, Aleksandar Nikolic, Jeff Wouters, Bartek Bielawski, and so many more&#8230; <a title="3rd German PowerShell Conference" href="http://www.powershell.de/konferenz" target="_blank">a few seats are still available</a>.

And if you like ISESteroids, then supporting the ISESteroids initiative is super easy: just make sure your company orders a really huge number of licenses! Just kidding, but it is true that license revenues fund all of this.

Hopefully we get a chance to meet at the PowerShell Conference!