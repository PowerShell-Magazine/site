---
title: Bartek Bielawski’s Favorite PowerShell Tips and Tricks
author: Bartek Bielawski
type: post
date: 2012-08-28T16:00:53+00:00
url: /2012/08/28/bartek-bielawskis-favorite-powershell-tips-and-tricks/
views:
  - 17050
post_views_count:
  - 2259
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I’m kind of guy who loves tiny little tricks. Tricks in PowerShell are almost like Easter eggs in the games of old times, very rewarding when you find them – but in PowerShell besides pure joy of being able to find something cool, you can move it to the next level, add it to your tool belt and use it every single day.

I’ve started using PowerShell 3 since first public build and tried to use it every single day: play with it, test things, break things, and also look for new tricks that were not there in v2. There are many cool additions known very well and mentioned by everybody who used this version for a while – some of them are focused on robust scripts, some on interactive use. All in all – we can be more productive with very little code on our side in v3. PowerShell team has put a lot of effort to give us new language features, new usability cmdlets, and significantly increased PowerShell coverage.

Let’s start with interactive work. You probably heard about simplified Where-Object and ForEach-Object syntax. Just in case you have not: this is ability to use Where-Object and ForEach-Object without a script block. You can obviously complain that it’s inconsistent, but if you are in shell and need to get data quickly – consistency is not your main focus. My first thought was: great for shell, bad for scripts – same as aliases. That said – I’ve been seeing people using it a lot, so why do I mention them? Because old habits die hard and even though new syntax gives some additional benefits, most of people just remove braces and $_. What do I mean? Let’s start with new Where-Object.

First of all, what looks like comparison operators are in fact parameters for your command. What it allows you to do is shortening comparer same way you can do that with other parameters. It won’t help you much with stuff like –eq or –in, but when we are talking about –match (-m) than you are really saving some time.

Another thing is the fact that arguments that are strings do not require quotes in 90% of scenarios. If you don’t have the characters that would trigger some parsing behaviour (like comma, semicolon, brackets) you can put the arguments without any quotes:

<pre class="brush: powershell; title: ; notranslate" title="">ls | ? Name -m unt
</pre>

Remember – this is for my eyes only. Or I should probably say – for my fingers. So even though it’s brief enough to become cryptic, in interactive work it should not matter. New ForEach-Object syntax can be used for two things: to quickly expand one of the properties (as an alternative you could use select –ExpandProperty since v2), or to call method on each element. Obviously, some methods do not have arguments, so regardless of the member type we end up with something as brief as:

<pre class="brush: powershell; title: ; notranslate" title="">gps notepad | % kill
</pre>

But that works just fine (or even better, in my opinion) once we start using methods that actually take some arguments. It’s getting crazy cool and sometimes make you want to use ForEach-Object (or its alias %) in situations, where you would avoid it in v2. Let’s look at good example of such method:

<pre class="brush: powershell; title: ; notranslate" title="">gwmi -List win32_share | % create $PSHOME POSH 0
gwmi win32_share | ? name -m po
</pre>

It’s almost like we could now call .NET methods the same way we call PowerShell functions, and I think this is very great thing to have. Behind the scenes it uses ValueFromRemainingArguments, but that’s again – neat trick to save some time and typing. 😉

Second trick or rather trick-combo is something anybody can use to make his objects shine with very brief and pretty awesome syntax. First part of this combo is syntax for new PSCustomObjects. Let’s say you are merging different information about single computer that you connect to, and need to create custom object as the end result. In v2 creating new objects is usually done using New-Object –Property (if order of parameters makes no difference) or Select-Object/Add-Member (if you actually care about the order). In v3 we have syntax that will share brevity of former, and ordered output of latter. Let’s build a function:

<pre class="brush: powershell; title: ; notranslate" title="">function Get-CompInfo {
	param (
	    [Parameter(
		ValueFromPipeline
	    )]
	    [string]$ComputerName
	)

	process {
	    $CIMSession = New-CimSession @PSBoundParameters
	    $PSDefaultParameterValues = @{
		'Get-CimInstance:CimSession' = $CIMSession
	    }
	    $CompInfo = Get-CimInstance -ClassName Win32_ComputerSystem
	    $OSInfo   = Get-CimInstance -ClassName Win32_OperatingSystem
	
	    [PSCustomObject]@{
		Name   = $CompInfo.Name
		User   = $CompInfo.UserName
		Domain = $CompInfo.Domain
		Model  = $CompInfo.Model
		OS     = $OSInfo.Caption
		SP     = $OSInfo.CSDVersion
	    }
	}
}
</pre>

As you can see it’s almost like it used be, only better… So we have first “hit”. Well, we actually got two – I smuggled something I would call “implicit splatting” – using $PSDefaultParameterValues to pass value for one of parameters to several commands without actually passing it at all. What’s next? A new syntax around Add-Member. Not only you can easily add few NoteProperties with it using simple hash table (that again &#8211; can be [ordered]), but you also can easily define custom type! Let’s change our function a bit to include both syntax elements:

<pre class="brush: powershell; title: ; notranslate" title="">function Get-CompInfo {
	param (
	    [Parameter(
		ValueFromPipeline
	    )]
	    [string]$ComputerName
	)

	process {
	    $CIMSession = New-CimSession @PSBoundParameters
	    $PSDefaultParameterValues = @{
		'Get-CimInstance:CimSession' = $CIMSession
	    }
	    $CompInfo = Get-CimInstance -ClassName Win32_ComputerSystem
	    $OSInfo = Get-CimInstance -ClassName Win32_OperatingSystem
	
	    [PSCustomObject]@{
		Name   = $CompInfo.Name
		User   = $CompInfo.UserName
		Domain = $CompInfo.Domain
		Model  = $CompInfo.Model
		OS     = $OSInfo.Caption
		SP     = $OSInfo.CSDVersion
	    } | Add-Member -TypeName My.Type -NotePropertyMembers ([ordered]@{
		Created = Get-Date
		Author  = $env:USERNAME
	    }) -PassThru
	}
}
</pre>

What if we would like to change the way my objects is shown quickly? In v3 we can again define default display PropertySet to avoid need for ps1xml format file to limit properties that are shown by default. Once we define this set in begin block, we can add it to our object before they leave our function:

<pre class="brush: powershell; title: ; notranslate" title="">function Get-CompInfo {
	param (
	    [Parameter(
		ValueFromPipeline
	    )]
	    [string]$ComputerName
	)

	begin {
	    $StandardMembers =
	    New-Object System.Management.Automation.PSPropertySet -ArgumentList DefaultDisplayPropertySet,
	    ([string[]]('Name','User','OS','SP'))
	    $MyTypeData = @{
		MemberType = 'MemberSet'
		Name       = 'PSStandardMembers'
		Value      = $StandardMembers
		TypeName   = 'MyType'
	    }
	
	}
	
	process {
	    $CIMSession = New-CimSession @PSBoundParameters
	    $PSDefaultParameterValues = @{
		'Get-CimInstance:CimSession' = $CIMSession
	    }
	    $CompInfo = Get-CimInstance -ClassName Win32_ComputerSystem
	    $OSInfo = Get-CimInstance -ClassName Win32_OperatingSystem
	
	    [PSCustomObject]@{
		Name   = $CompInfo.Name
		User   = $CompInfo.UserName
		Domain = $CompInfo.Domain
		Model  = $CompInfo.Model
		OS     = $OSInfo.Caption
		SP     = $OSInfo.CSDVersion
	    } | Add-Member -TypeName My.Type -NotePropertyMembers ([ordered]@{
		Created = Get-Date
		Author  = $env:USERNAME
	    }) -PassThru | Add-Member @MyTypeData -PassThru
	}
}
</pre>

And last but not least – something that will be useful both for custom, user-defined types and for modifying behaviour of existing types – new syntax for Update-TypeData:

<pre class="brush: powershell; title: ; notranslate" title="">Update-TypeData -MemberType ScriptMethod -Value {
    @"
    Computer {0}, used by {1}, running {2} with {3}.
    Inventory created: {4:G} by {5}.
"@ -f @(
        $this.Name,
        $this.User,
        $this.OS,
        $this.SP,
        $this.Created,
        $this.Author)

} -MemberName Summary -TypeName My.Type
</pre>

See? Now our need to use ps1xml files got limited to some really advanced situations only, and logic and readability of functions/ scripts got way better. This finalizes this tricks-combo. J

The last trick I would like to share is something that might be useful to people who would like to ignore existence of WQL. The problem with this language is that unlike most of PowerShell commands – it will require more SQL-like syntax for wildcards. But wouldn’t it be nice to have something that would translate them for us? And now, in v3, we actually got tool to do that:

<pre class="brush: powershell; title: ; notranslate" title="">([Management.Automation.WildcardPattern]@'
Some*g To Trans?ate - With _escaping_ %)
'@).ToWql()
</pre>

All you need to do is create the logic for your function that would translate any pattern specified by user (using normal wildcards) to pattern. Such logic could look like this:

<pre class="brush: powershell; title: ; notranslate" title="">$query = 'Name -like "*foo*" -AND Bla -like "*bar*"'

$tokens = [Management.Automation.PSParser]::Tokenize($query, [ref]$null)

$FilterElements = @()

foreach ($token in $tokens) {
    switch ($token.Type) {
        CommandParameter {
            # stuff like: -like and -and... replace with same w/o '-'
            $FilterElements += $token.Content -replace '^-'
        }

        String {
            # assume this is pattern we are after... Need to keep quotes, too.
            $FilterElements += "'$(
                ([Management.Automation.WildcardPattern]$Token.Content).ToWql()
                )'"
        }
    
        Default {
            # no changes needed...
            $FilterElements += $token.Content
        }
    }
}

$FilterElements -join " "
</pre>

And that’s third trick I would like to share. There are a lot more to come in next version of Windows PowerShell, good luck with finding them. And don’t forget to share them, once you find one!