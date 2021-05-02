---
title: Using custom argument completers in PowerShell 3.0
author: Jan Egil Ring
type: post
date: 2012-11-29T17:00:50+00:00
url: /2012/11/29/using-custom-argument-completers-in-powershell-3-0/
categories:
  - tabexpansion2
  - Module Spotlight
tags:
  - Modules
  - tabexpansion2

---
Custom argument completers is a new feature in PowerShell 3.0 which makes it possible to define your own argument values for parameters. Let us use Set-ExecutionPolicy as an example.

Due to the new IntelliSense feature in Windows PowerShell Integrated Scripting Environment (ISE) 3.0, we get a list of available parameters when we type the name of a cmdlet, press space and type a dash:

![](/images/image001.jpg)

As you might have noticed, this also works for the parameter values on cmdlets where the cmdlet author has implemented this feature:

![](/images/image0021.jpg)

PowerShell MVP Tobias Weltner has written an excellent [article][1] describing how you can define custom parameter arguments using the new custom argument completer functionality in PowerShell 3.0.

As an example, wouldn&#8217;t it be nice if you could get a list of all computers from Active Directory when you hit space after the –ComputerName parameter on any cmdlet or function which uses this parameter?

This is fully possible using a custom argument completer. First, let&#8217;s retrieve a string array of all computers from Active Directory. Because we do not want to be dependent on any modules or snap-ins like Microsoft&#8217;s ActiveDirectory module or Quest&#8217;s PowerShell Commands for Active Directory, we use ADSI to retrieve the data:

<pre class="brush: powershell; title: ; notranslate" title="">$searcher = [adsisearcher]"(&(objectClass=Computer)(operatingSystem=Windows Server*))"
$searcher.PropertiesToLoad.Add("cn") | Out-Null
$searcher.PropertiesToLoad.Add("operatingsystem") | Out-Null
$searcher.FindAll()
</pre>

The above example will retrieve all computer accounts from the Active Directory domain your computer is a member of, where the attribute “operatingSystem” begins with the word “Windows Server”.

Next, we add the code that retrieves the data we want to use into a custom completion handler:


    $Completion_ComputerName = {
        param($commandName, $parameterName, $wordToComplete, $commandAst, $fakeBoundParameter)
        $searcher = [adsisearcher]"(&(objectClass=Computer)(operatingSystem=Windows Server*))"
        $searcher.PropertiesToLoad.Add("cn") | Out-Null
        $searcher.PropertiesToLoad.Add("operatingsystem") | Out-Null
        $searcher.FindAll() | ForEach-Object { New-Object -TypeName pscustomobject -Property @{
            ComputerName = ($_.properties)["cn"].Item(0)
            OperatingSystem = ($_.properties)["operatingsystem"].Item(0)
            }
        } | Sort-Object ComputerName | ForEach-Object {
            New-Object System.Management.Automation.CompletionResult $_.ComputerName, $_.ComputerName, 'ParameterValue', ('{0} ({1})' -f $_.ComputerName, $_.OperatingSystem)
        }
    }
At last we need to add the handler into built-in tabexpansion2 function:

```
if (-not $global:options) {
	$global:options = @{CustomArgumentCompleters = @{};NativeArgumentCompleters = @{}}
}
$global:options['CustomArgumentCompleters']['ComputerName'] = $Completion_ComputerName

$function:tabexpansion2 = $function:tabexpansion2 -replace 'End\r\n{','End { if ($null -ne $options) { $options += $global:options} else {$options = $global:options}
```

Now we can test if our custom completer works by using any cmdlet with a –ComputerName parameter, for example Get-Service:

![](/images/image003.jpg)

Here we can see IntelliSense providing computer names from Active Directory. Also note that the operating system is listed in parenthesis. Instead of retrieving the values from Active Directory you could also get them from a text file, CSV-file, SQL database, or any other data store you can access from PowerShell.

As you can see, the new custom argument completion feature opens up for a lot of interesting use cases.

[1]: http://www.powertheshell.com/dynamicargumentcompletion/