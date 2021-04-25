---
title: '#PSTip Script sharing using Dropbox'
author: David Moravec
type: post
date: 2012-10-29T18:00:13+00:00
url: /2012/10/29/pstip-script-sharing-using-dropbox-2/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When I was asked to provide some PowerShell tips, I decided to look at the first possible place&#8211;my _**$profile**_. When I started working with PowerShell few years ago I was looking for solution to have scripts available on all of my computers. Then I found easy solution&#8211;Dropbox. At the moment I have this structure:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Show-Tree Dropbox:\PowerShell -Depth 1
Dropbox:\PowerShell
├──Books
├──Mercurial
├──My
├──Papers
├──Presentations
├──Profile
├──pse_1.0.beta.x86
├──Scripts
├──Trainings
└──v3
</pre>

You can see that for showing this structure, I&#8217;ve used Show-Tree function, a part of [PowerShell Community Extension][1]s. If you haven’t checked that one out yet, it’s time for it now. It has really a lot of very useful functions.

First line of my _**$PROFILE.AllUsersAllHosts**_ is:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Content $PROFILE.AllUsersAllHosts | Select -First 1
New-PSDrive -Name Dropbox -PSProvider FileSystem -Root "c:\Documents and Settings\Moravec\My Documents\Dropbox" | Out-Null
</pre>

This allows me to have available anything I am currently working on (plus all my favorite books). I&#8217;ve also modified the _$PSModulePath_ automatic variable:

<pre class="brush: powershell; title: ; notranslate" title="">$env:PSModulePath += ';Dropbox:\PowerShell\Profile'</pre>

Just to ensure that when I add anything to Dropbox module folder, I’ll have it everywhere. For the same reason I created new “profile” script which I use for synchronizing my frequently using scripts and functions:

<pre class="brush: powershell; title: ; notranslate" title="">. Dropbox:\PowerShell\Profile\profile_Dropbox.ps1</pre>

I don’t use the same profile on all computers. On my home computer it’s not necessary to call, for example, a module I use for work with Configuration Manager. I know that I will never connect to my infrastructure from a personal netbook.

[1]: http://pscx.codeplex.com/