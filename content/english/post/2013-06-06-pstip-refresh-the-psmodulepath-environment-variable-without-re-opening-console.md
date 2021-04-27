---
title: '#PSTip Refresh the PSModulePath environment variable without re-opening console'
author: Ravikanth C
type: post
date: 2013-06-06T18:00:00+00:00
url: /2013/06/06/pstip-refresh-the-psmodulepath-environment-variable-without-re-opening-console/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
A co-worker had asked me a question about why the SQLPS module was not available for importing immediately after an automated install of SQL using a PowerShell script. Within this script, he was installing SQL as step 1 and then in step 2, he needs to use SQLPS module for setting some SQL configuration. However, when the script came to step 2, it complained that the SQLPS module was not found.

If you have worked on SQL PowerShell module, you may be aware of the fact that they store the module in Program Files folder for SQL Server and they update the PSModulePath system environment variable. But, this change won&#8217;t be available to PowerShell unless you reopen the PowerShell console. In a sequential execution flow, this is not an option.

Here is how we solved the problem:

<pre class="brush: powershell; title: ; notranslate" title="">$env:PSModulePath = [System.Environment]::GetEnvironmentVariable("PSModulePath","Machine")
</pre>