---
title: '#PSTip Send the last command executed to clipboard'
author: Ravikanth C
type: post
date: 2012-11-06T19:00:42+00:00
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

When I start writing a script, I generally start at the shell and make sure the logic I am working on holds good. Also, when writing blog posts, I tend to use the console &#8211; either powershell.exe or powershell ISE console &#8211; and then copy the commands into a blog post.

So, generally, I end up copying the last command I executed to either a blog post or a script. So, here is a small snippet I use to achieve that!

<pre class="brush: powershell; title: ; notranslate" title="">(Get-History)[-1].commandline | clip
</pre>

Simple! The trick to get the last executed command is to use an array index -1 which means the last item in the array. Now, all I need to do is put this in a simple function and put it in my profile for easy access:

<pre class="brush: powershell; title: ; notranslate" title="">Function Copy-LastCommand {
 (Get-History)[-1].commandline | clip
}
</pre>

<div>
</div>