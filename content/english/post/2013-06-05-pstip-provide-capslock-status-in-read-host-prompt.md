---
title: '#PSTip Provide CapsLock status in Read-Host prompt'
author: Ravikanth C
type: post
date: 2013-06-05T18:00:00+00:00
url: /2013/06/05/pstip-provide-capslock-status-in-read-host-prompt/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In Windows, you get a message about the CapsLock status when logging on to Windows OS and with CapsLock on. You see this kind of experience in many other applications including web applications.

A similar experience, if not the same, can be provided in a simple manner when asking for password input using _Read-Host_ cmdlet in a script.

<pre class="brush: powershell; title: ; notranslate" title="">Read-Host -Prompt "Enter Password $(if([console]::capslock){'(CapsLock is ON)'})"
</pre>

Here is how it looks:

![](/images/capslock.png)

