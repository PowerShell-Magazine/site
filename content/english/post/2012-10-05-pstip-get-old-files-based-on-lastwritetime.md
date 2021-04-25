---
title: '#PSTip Get old files based on LastWriteTime'
author: Shay Levy
type: post
date: 2012-10-05T18:00:41+00:00
url: /2012/10/05/pstip-get-old-files-based-on-lastwritetime/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 and above

There are many ways to get the age of a file. The most common way is to subtract the file&#8217;s LastWriteTime from the current time:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $now = Get-Date
PS&gt; $prof = Get-ChildItem $PROFILE
PS&gt; ($now - $prof.LastWriteTime).Days
280
</pre>

Or by using the Subtract method:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $now.Subtract($prof.LastWriteTime).Days
280
</pre>

There is another way to get the information using a  less known method of piping a file system object to the New-TimeSpan cmdlet:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ($prof | New-TimeSpan).Days
280
</pre>

The Start parameter of the New-TimeSpan cmdlet has a LastWriteTime parameter Alias which automatically bind the value of the incoming file system object

```
PS> (Get-Command New-TimeSpan).Parameters.Start

Name            : Start
ParameterType   : System.DateTime
ParameterSets   : {[Date, System.Management.Automation.ParameterSetMetadata]}
IsDynamic       : False
Aliases         : {LastWriteTime}
Attributes      : {System.Management.Automation.AliasAttribute, Date}
SwitchParameter : False
```

Here&#8217;s an example of all DLL file&#8217;s age in PowerShell&#8217;s installation directory

```
PS> $age = @{Name='Age(Days)'; Expression={($_ | New-TimeSpan ).Days }}
PS> Get-ChildItem $PSHOME -Filter *.dll | Select-Object Name,$age

Name                  Age(Days)
----                  ---------
PSEvents.dll                 68
pspluginwkr-v3.dll           68
pspluginwkr.dll             121
pwrshmsg.dll                 68
pwrshsip.dll                 68
```

