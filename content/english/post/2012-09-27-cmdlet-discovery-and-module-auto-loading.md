---
title: '#PSTip Cmdlet Discovery and Module auto-loading'
author: Shay Levy
type: post
date: 2012-09-27T18:00:36+00:00
url: /2012/09/27/cmdlet-discovery-and-module-auto-loading/
views:
  - 10709
post_views_count:
  - 2414
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In PowerShell 3.0, we no longer have to import modules manually to use cmdlets. When we run a cmdlet, Windows PowerShell imports the module that contains the command automatically. In addition, the Get-Command cmdlet now returns all cmdlets installed on the system, even if they are not in the current session.

We can enable, disable, and configure automatic importing of modules by using the _$PSModuleAutoLoadingPreference_ preference variable. Valid values for this variable are: None, ModuleQualified, or All.

| All             | Module auto-loading works for all commands. To import a module, get or use any command in the module. |
| --------------- | ------------------------------------------------------------ |
| None            | Module auto-loading is turned off. To import a module, use the Import-Module cmdlet. |
| ModuleQualified | Modules are imported automatically only when a user uses the module-qualified name of a command. For example, if the user types “MyModule\MyCommand”, Windows PowerShell imports the MyModule module. |

Alternatively, we can limit the command search scope and get only commands in the current session with the -ListImported switch parameter.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command &lt;CommandName&gt; -ListImported
</pre>