---
title: '#PSTip Getting system modules only'
author: Shay Levy
type: post
date: 2012-08-22T18:00:31+00:00
url: /2012/08/22/pstip-getting-system-modules-only/
views:
  - 5449
post_views_count:
  - 987
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks
---
When we use the Get-Module -ListAvailable command to list installed modules, we get a list of modules from two default module locations: one for the system and one for the current user.

By default, the current user modules are located under the current user&#8217;s Documents (MyDocuments) folder and system modules reside under the PowerShell installation directory. To view the default module locations, type:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $env:PSModulePath
</pre>

The value is hard to read and is displayed in one long line delimited by a semi colon. To display the paths, one on its own line, type:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $env:PSModulePath -split ';'
</pre>

Each module info object returned by Get-Module has a Path property. We can use the Path to programmatically determine if a module is a user module or a system module. For system modules, we can check if the module path starts with the path of Windows PowerShell installation directory:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Module -ListAvailable | Where-Object {$_.Path -like "$PSHOME*" }
</pre>

To produce a list of user modules, we can simply negate the filter using the -NotLike operator:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt;  Get-Module -ListAvailable | Where-Object {$_.Path -notlike "$PSHOME*" }
</pre>