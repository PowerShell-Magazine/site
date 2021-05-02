---
title: '#PSTip Get the AD site name of a computer'
author: Shay Levy
type: post
date: 2013-04-23T18:00:00+00:00
url: /2013/04/23/pstip-get-the-ad-site-name-of-a-computer/
categories:
  - Tips and Tricks
  - Active Directory
tags:
  - Active Directory
  - Tips and Tricks

---
There are a few ways to get the site a computer is a member of. In .NET we can use the [_ActiveDirectorySite_ class][1].

<pre class="brush: powershell; title: ; notranslate" title="">[System.DirectoryServices.ActiveDirectory.ActiveDirectorySite]::GetComputerSite().Name
</pre>

This can be extremly usefull if you want to base a script on the value of the computer&#8217;s site. Sometimes, however, you&#8217;ll want to query the site of remote computer. Unfortunately, the _ActiveDirectorySite_ class doesn&#8217;t allow that. One way to get the information is to query the _DynamicSiteName_ registry value of the remote machine. The current site information is cached in the registry of a given machine under (HKLM:\SYSTEM\CurrentControlSet\services\Netlogon\Parameters).

Another way would be using the [_nltest_][2] command line utility

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; nltest /server:server1 /dsgetsite
Default-First-Site-Name
The command completed successfully
</pre>

If the command completed successfully, we&#8217;ll have the site name in the first line. The last step is to wrap this into a function so we can reuse it later on.

```
function Get-ComputerSite($ComputerName)
{
	$site = nltest /server:$ComputerName /dsgetsite 2&>$null
	if($LASTEXITCODE -eq 0){ $site[0] }
}

PS> Get-ComputerSite server1

Default-First-Site-Name
```

[1]: http://msdn.microsoft.com/en-us/library/5ka2e3ec.aspx
[2]: http://technet.microsoft.com/en-us/library/cc731935(v=ws.10).aspx