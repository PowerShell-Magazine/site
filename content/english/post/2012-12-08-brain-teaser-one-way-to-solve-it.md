---
title: Brain teaser – one way to solve it
author: Shay Levy
type: post
date: 2012-12-08T17:00:33+00:00
url: /2012/12/08/brain-teaser-one-way-to-solve-it/
categories:
  - Brainteaser
tags:
  - Brainteaser

---
This is the command I had initially in mind when writing the [last modified time teaser][1]:

<pre class="brush: powershell; title: ; notranslate" title="">ls $pshome\powershell.exe|date
</pre>

I also had this one, which is shorter:

<pre class="brush: powershell; title: ; notranslate" title="">ps -id $pid|gi|date
</pre>

But I specifically asked for powershell.exe and the above might report the last modified time of hosts other than powershell.exe (e.g ISE). Some of the answers also used wildcards for the file name. It certainly shortens the command but may introduce false results if similar files were added to the PowerShell folder.

Anyway, the trick to make it work without having to refer to the _LastWriteTime_ property is to pipe a file system object to the _Get-Date_ cmdlet. Why does that work?

The _Date_ parameter of the _Get-Date_ cmdlet accepts pipeline input by property name (the parameter attribute _ValueFromPipelineByPropertyName_ is set to _$true_). The _Date_ parameter also defines a _LastWriteTime_ alias. This means that the value of incoming objects that have a _Date_ property (or a _LastWriteTime_ property), is assigned to the _Date_ parameter.

```
PS> (Get-Command Get-Date).Parameters.Date

Name            : Date
ParameterType   : System.DateTime
ParameterSets   : {[__AllParameterSets, System.Management.Automation.ParameterSetMetadata]}
IsDynamic       : False
Aliases         : {LastWriteTime}
Attributes      : {__AllParameterSets, System.Management.Automation.AliasAttribute}
SwitchParameter : False
```

And why _date_ and not _Get-Date_? When PowerShell search for a command, if the command was not found it tries to prepend it with the default _Get_ verb.

So, if PowerShell can&#8217;t find a command named _&#8216;date&#8217;_, it will try a second time with _Get-Date_.

You can see this in action if you execute any Get command without the verb. For instance, _service_ vs. _Get-Service_, and so on.

That&#8217;s it. I hope you liked the challenge, I know I did!

This time the winner is **Jakub Jareš**. Jakub, I hope you had a good night sleep the other day, you nailed it! Head on to Jakub&#8217;s blog, he wrapped up his work on the teaser on his blog: http://powershell.cz/2012/12/07/last-modified-date-brainteaser

Sorry we don&#8217;t have any giveaways this time but you&#8217;ve got our respect 🙂

[1]: /2012/12/04/get-the-last-modified-date-and-time/ "Get the last modified date and time"