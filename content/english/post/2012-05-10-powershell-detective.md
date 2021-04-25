---
title: PowerShell Detective
author: Bartek Bielawski
type: post
date: 2012-05-10T18:00:45+00:00
url: /2012/05/10/powershell-detective/
views:
  - 9176
post_views_count:
  - 1893
categories:
  - How To
tags:
  - How To

---
I guess when you use PowerShell; you very often don’t care how it works. As long as it does what you asked it to do – you are happy. Your job is done on time in consistent, repetitive manner, and you have time to spend with family and friends. But at times you may start to wonder, how it works? When you ask this question, neither –Verbose nor –Debug can answer. It’s either unexpected behaviour of cmdlet, or some magic that happens regardless what documentation clearly states. The part of you which likes riddles, and likes them solved, kicks in. And you begin to dig into the environment. And, of course – you can hire Google-fu, but at times it won’t get you far.

When that happens, it’s the good moment to start PowerShell and take a closer look at two cmdlets: Get-TraceSource and Trace-Command.

### Form a question

In order to get the right answer – you first need to be sure you asked the question right. That’s why Get-TraceSource is very important part of this puzzle. When you launch it without any parameters you will get the list of possible trace names. Each name is accompanied by description that can be helpful at times, but not always. This cmdlet itself lets you look for particular names using wildcards. Let’s say you want to look deeper into the way parameters are bound by commands. It would be as simple as:

<pre>PS&gt; Get-TraceSource -Name *param*</pre>

One thing to keep in mind: some sources are loaded **after** command is used. For example, try to find any Native* sources before running any native command (like ping, ipconfig, etc):

![](/images/PSDetective1.png)

If -Name is not enough then &#8212; as usually in PowerShell – resulting output of Get-TraceSource can be easily filtered. Just ask for trace source where description matches pattern you are after. And because not all names are very descriptive, or rather – some descriptions are more informative than just $_.Name.Split() – it should help you get to correct source names.

Once you find out what the possible trace name is (you can pick up few, Trace-Command accepts both arrays and wildcards) next step is to actually ask your question and see if this really gives you an answers or just generates several new questions instead.

### Ask your question

When you got that far you are probably tempted to ask as general question as possible. But you should be careful – Trace-Command can be so verbose in its output, that it can kill your console at times. Also, specifying &#8216;*’ as the name of the trace will seldom be a good idea.

This is moment where two “gotchas” may stop you for a moment or two. First of all – PowerShell is very bad if you specify wrong source name. You won’t get error, warning, suggestion to check valid sources – instead you will get no trace messages at all. If you are confident those messages should show up, you will probably notice a typo eventually. If you are not so sure – you may assume that you just guessed wrong source and look for another ‘suspect’.

Next thing is [listener][1] that will be target of you trace. If you want to actually see anything you have to either enable –PSHost switch, or select –FilePath. If you haven’t, nothing will show up. Once you got both right – you will get some output that may (or may not) give you clue on what is actually going on:

![](/images/PSDetective2.png)

If the output is just too much to read on the screen – it’s worth considering use of –FilePath instead of –PSHost. And if you want to inject previously run command into –Expression, just use #<pattern>[TAB] – it will get you there promptly.

But, honestly, do you really want to do all that? Copy and paste trace names ‘just in case’? Remember to include listener? And last but not least, inject code using some tricks each time instead of putting it in curly braces and just pass it to tracing command using pipe? Or maybe you do not care about command output now that you have seen it already?

So, why not build simple wrapper that would prevent us from doing all that?

### Questions generator

I’ve decided long ago to do exactly that: create simple wrapper around Trace-Command that would:

  * validate ‘Name’ parameter using Get-TraceSource
  * use either PSHost or FilePath (if it’s specified)
  * sent output to $null if –Quiet option is selected
  * pick up expression from pipeline, to support easy injection of previously run commands

I also decided that I’m fine with chopping off some other options that I hardly use (but it should not be hard to implement them later) like listener options. No more silence when you specify wrong name and no more silence when you forget to select listener. Also, pure traces without some commands output mixed here and there. So here you go, early beta of PowerShell Detective module. 😉

```powershell
function Trace-Expression {
  [CmdletBinding(DefaultParameterSetName = 'Host')]
  param (
    # ScriptBlock that will be traced
	[Parameter(
		ValueFromPipeline = $true,
		Mandatory = $true,
		HelpMessage = 'Expression to be traced'
	)]
	[ScriptBlock]$Expression,

	# Name of the Trace Source(s) to be traced
    [Parameter(
        Mandatory = $true,
        HelpMessage = 'Name of trace, see Get-TraceSource for valid values'
    )]
    [ValidateScript({
        Get-TraceSource -Name $_ -ErrorAction Stop
    })]
    [string[]]$Name,

	# Option to leave only trace information
	# without actual expression results.
	[switch]$Quiet,

	# Path to file. If specified - trace will be sent to file instead of host.
    [Parameter(ParameterSetName = 'File')]
    [ValidateScript({
        Test-Path $_ -IsValid
    })]
    [string]$FilePath
  )

  begin {
    if ($FilePath) {
        # assume we want to overwrite trace file
        $PSBoundParameters.Force = $true
    } else {
        $PSBoundParameters.PSHost = $true
    }
    if ($Quiet) {
        $Out = Get-Command Out-Null
        $PSBoundParameters.Remove('Quiet') | Out-Null
    } else {
        $Out = Get-Command Out-Default
    }
  }

  process {
    Trace-Command @PSBoundParameters | & $Out
  }
}

PS> New-Alias -Name tre -Value Trace-Expression
PS> Export-ModuleMember -Function * -Alias *
```

### Answer found

I like to know what is going on behind the scenes. That’s why I use Trace-Command quite often. It helped me very much in the past when I had problem with understanding how Foreach-Object works. What was the issue?

Well, I had no idea why …

```powershell
PS> 1..5 | ForEach-Object { 'begin' } { "process: $_" } { 'end' }
```

… works and why both –Begin and –End parameters are positional. PowerShell help will tell you that this is not the case. If you use Trace-Expression, you will soon find out that documentation is right, but is not precise in describing how cmdlet as a whole works. It misses very important piece: parameter –Process will take ValueFromRemainingArguments (same way $args does in functions without param block) and if there are more than one – it will try to distribute them between begin, process and end block. First part can be easy traced:

![](/images/PSDetective3.png)

I was not able to find second part (distribution of script blocks) with Trace. But you can guess what is going on because you can get the same result if you make sure all script blocks will be passed to -Process parameter:

```powershell
PS> 1..5 | ForEach-Object -Process { 'begin' }, { "process: $_" }, { 'end' }
```

I blogged [about this][2] but found the answer in Bruce Payette’s PowerShell in Action few weeks later. But that incident made me confident that if there are some PowerShell behaviours I don’t understand immediately, then using \*Trace\* command may provide an explanation. It helped me when I was trying to find out why function that had no Get-Process command in it was complaining about wrong syntax used with this cmdlet.

Let’s reproduce this issue first:

```powershell
<pre>function Get-Foo {
   param (
    [Parameter(ValueFromPipeline = $true)]
    [string]$Bar
  )

  $Foo = 2

  process {
    $Bar
  }
}
```

More experienced users will probably spot an issue immediately in this code, but real example was more complex than this one. What will happen if we use it?

![](/images/PSDetective4.png)

This was starting point. Obviously, you won’t find any Get-Process in my code, so where did this error come from? At first, I’ve traced everything to understand why it works like that. Eventually I found root cause of this behaviour.

This is combination of two things: how keywords behave depending on the context and what PowerShell does if a given command doesn’t exist. PowerShell keywords are context-aware. That makes things like ‘foreach’ alias possible, but here, it complicates things. Because ‘process’ is used in the wrong context it’s no longer a keyword, PowerShell will assume its command:

![](/images/PSDetective5.png)

But, how did we get an error from **Get-**Process? We won’t find alias or any other command named ‘process’ with Get-Command. That’s another feature that in this scenario becomes an issue:

![](/images/PSDetective6.png)

So not only our keyword was behaving like a command, it was also prepended with get- to make our life easier, something that makes things like ‘date’ instead of ‘Get-Date’ possible.

So remember, whether it’s binding of arguments, ETS, converting types – whatever seems suspicious at first may become obvious once you employ detective inside you, together with two tools that no PowerShell detective can live without: Trace-Command and Get-TraceSource.

[1]: http://msdn.microsoft.com/en-us/library/4y5y10s7.aspx
[2]: http://becomelotr.wordpress.com/2011/05/22/magic-of-foreach-object/