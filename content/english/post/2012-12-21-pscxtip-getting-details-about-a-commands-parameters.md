---
title: '#PSCXTip Getting details about a command’s parameters'
author: Keith Hill
type: post
date: 2012-12-21T19:00:55+00:00
url: /2012/12/21/pscxtip-getting-details-about-a-commands-parameters/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The parameters for a PowerShell command can be quite complex. Some parameters are only valid in certain parameter sets. Parameters have types, may be mandatory or optional, can have aliases and may or may not support pipeline input. The standard PowerShell mechanism to inspect a command’s parameters is to look at the command’s help page. This can be informative. See the output for a few of the parameters to _Get-Process_:

```
-Id <Int32[]>
    Specifies one or more processes by process ID (PID). To specify multiple IDs,
    use commas to separate the IDs.
    To find the PID of a process, type "get-process".

    Required?                    True
    Position?                    Named
    Default value
    Accept pipeline input?       true (ByPropertyName)
    Accept wildcard characters?  False

-Name <String[]>
    Specifies one or more processes by process name. You can type multiple process
    names (separated by commas) and use wildcard characters. The parameter name
    ("Name") is optional.

    Required?                    False
    Position?                    1
    Default value
    Accept pipeline input?       true (ByPropertyName)
    Accept wildcard characters?  true
```

What the help file doesn’t show is that these two parameters are in different parameter sets and can’t be used in the same command invocation. Also, the help isn’t always in lock-step with the actual parameters based on the way the help files are created.

The ultimate source of parameter information is in the _CmdletInfo_ object for the command. We can see some of this information using PowerShell’s _Get-Command_ cmdlet:

```
PS> Get-Command Get-Process -Syntax

Get-Process [[-Name] <string[]>] [-ComputerName <string[]>] [-Module]
[-FileVersionInfo] [<CommonParameters>]

Get-Process -Id <int[]> [-ComputerName <string[]>] [-Module] [-FileVersionInfo]
[<CommonParameters>]

Get-Process -InputObject <Process[]> [-ComputerName <string[]>] [-Module]
[-FileVersionInfo] [<CommonParameters>]
```

This at least shows us that there are different parameter sets and that the _Id_ and _Name_ parameters are in different parameter sets. We even get type information for the parameters. However there is no information about accepting pipeline input nor is there any information on parameter aliases.

There is a command in the <a title="PowerShell Community Extensions" href="http://pscx.codeplex.com" target="_blank">PowerShell Community Extensions</a> (2.1 and 3.0) that displays rich parameter information in an easy to read format:

```
PS> Get-Parameter Get-Process
Command: Microsoft.PowerShell.Management/Get-Process
Set:     Name

Name            Aliases      Position Mandatory Pipeline ByName Provider        Type
----            -------      -------- --------- -------- ------ --------        ----
ComputerName    {Cn, co, ... Named    False     False    True   All             String[]
FileVersionInfo {FV, FVI,... Named    False     False    False  All             SwitchPara...
Module          {m, mo, m... Named    False     False    False  All             SwitchPara...
Name            {ProcessN... 0        False     False    True   All             String[]

Command: Microsoft.PowerShell.Management/Get-Process
Set:     Id
Name            Aliases      Position Mandatory Pipeline ByName Provider        Type
----            -------      -------- --------- -------- ------ --------        ----
ComputerName    {Cn, co, ... Named    False     False    True   All             String[]
FileVersionInfo {FV, FVI,... Named    False     False    False  All             SwitchPara...
Id              {PID, id}    Named    True      False    True   All             Int32[]
Module          {m, mo, m... Named    False     False    False  All             SwitchPara...

Command: Microsoft.PowerShell.Management/Get-Process
Set:     InputObject
Name            Aliases      Position Mandatory Pipeline ByName Provider        Type
----            -------      -------- --------- -------- ------ --------        ----
ComputerName    {Cn, co, ... Named    False     False    True   All             String[]
FileVersionInfo {FV, FVI,... Named    False     False    False  All             SwitchPara...
InputObject     {in, inp,... Named    True      True     False  All             Process[]
Module          {m, mo, m... Named    False     False    False  All             SwitchPara...
```

As you can see, there’s much more usable information available from the _Get-Parameter_ command.

**Note**: There are many more useful PowerShell Community Extensions (PSCX) commands. If you are interested in this great community project led by PowerShell MVPs <a title="Keith Hill's blog" href="http://rkeithhill.wordpress.com" target="_blank">Keith Hill</a> and <a title="Oisin Grehan's blog" href="http://www.nivot.org" target="_blank">Oisin Grehan</a>, give PSCX a try at <http://pscx.codeplex.com>.