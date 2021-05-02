---
title: '#PSTip How to prevent script execution for non-admin users'
author: Shay Levy
type: post
date: 2013-09-11T18:00:26+00:00
url: /2013/09/11/pstip-how-to-prevent-script-execution-for-non-admin-users/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 4.0 or above.

The _#Requires_ statement allows us to prevent a script from running without the required elements. For example, we can specify a minimum version of PowerShell that the script requires.

<pre class="brush: powershell; title: ; notranslate" title="">#Requires -Version 3
</pre>

Other elements can be a _PSSnapin, Module_, and a specific _ShellId_ (see the <a title="about_Requires help topic" href="http://technet.microsoft.com/en-us/library/hh847765(v=wps.630).aspx" target="_blank">about_Requires</a> topic). PowerShell 4.0 added another useful prerequisite: _RunAsAdministrator_. When this switch parameter is added to your _Requires_ statement, it specifies that the Windows PowerShell session in which you are running the script must be started with elevated user rights (in other words, by using the &#8216;Run as Administrator&#8217; option).

The following statements require the _ActiveDirectory_ module and elevated user rights. If the _ActiveDirectory_ module is not in the current session, PowerShell will import it. If the module cannot be imported, PowerShell throws a terminating error. If you haven&#8217;t used &#8220;_Run as Administrator_&#8221; option to start your PowerShell session, you will get an error as well.

<pre class="brush: powershell; title: ; notranslate" title="">#Requires -Modules ActiveDirectory
#Requires -RunAsAdministrator
</pre>