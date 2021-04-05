---
title: PowerShell Tools for the Advanced Use Cases, part 1
author: "David O'Brien"
type: post
date: 2015-10-12T16:30:17+00:00
url: /2015/10/12/powershell-tools-for-the-advanced-use-cases-part-1/
views:
  - 22619
post_views_count:
  - 3380
categories:
  - How To
  - DevOps
tags:
  - How To
  - DevOps

---
### Posts in this series

  1. PowerShell Tools for the Advanced Use Cases, part 1 (this article)
  2. [PowerShell tools for the advanced use cases – part 2][1]
  3. [PowerShell tools for the advanced use cases – part 3][2]

In the world of DevOps it’s all about automation&#8211;automated installation of applications, automated configuration of applications, all automated. Infrastructure as Code.

The thing is a lot of people think it is enough to just write a script and off you go. That is not enough. Code has to be tested, requirements be checked, or artefacts be built.

This article will introduce a couple of tools especially written for/in PowerShell or that can be used in combination with PowerShell that will make your production code more robust/reliable and are ultimately easy to implement into a Continuous Integration pipeline.

I am not going to talk about “Why use Source Control” here, as I am now assuming that all of you are already using some sort of Version and Source Control like Git(hub/lab), VSO, TFS, or even Mercurial. I am going to talk about some tools that are often ignored or a lot of people don’t even know exist.

### PSScriptAnalyzer

Are you a member of a team? Do you share PowerShell modules inside of this team or have multiple team members work on the same code once in a while? Do you share your code to the community&#8211;via Git(hub/lab) or the <a href="http://www.powershellgallery.com" target="_blank">PowerShell Gallery</a>? Or, do you code for yourself but still want to follow some accepted best practises?

#### Enter PSScriptAnalyzer

If you have done any work on Linux or with other scripting/development languages before then you might have come across _linters_. Pylint, yaml-lint, puppet-lint, json-lint, rubocop, these are all examples of very mature style guide checks for several languages that are not born on Windows, but also apply to use case scenarios on Windows, so do check them out.

However, none of them checks PowerShell style. There were some tools around in the past, but now Microsoft has released the new version of PSScriptAnalyzer to their PowerShell Gallery.

```powershell
Find-Module -Name PSScriptAnalyzer | Select-Object Version, Name, Description | Format-List *
```


![](/images/advtools11.png)

The module is hosted on the PowerShell Gallery and can easily be installed via PowerShellGet: _Install-Module -Name PSScriptAnalyzer_

The PSScriptAnalyzer module comes with two commands:

![](/images/advtools12.png)

Using the PSScriptAnalyzer is more than simple. Call the Invoke-ScriptAnalyzer cmdlet and point it either to a folder with PowerShell scripts in it or to a PowerShell module and it will analyse your code.

I pointed it at my <a href="https://gitlab.com/dobrien/GithubConnect" target="_blank">GithubConnect</a> module (an older version of it) and got these results:

![](/images/advtools13.png)

With that one line my whole module has been tested against 38 built-in rules (Module Version 1.1.0, 10/10/2015) and comes back with two errors and it exactly points me to the file and line I have to go and investigate.

If you want to see all the built-in rules, call _Get-ScriptAnalyzerRule_ and check each rule’s description.

#### How does PSScriptAnalyzer work?

Pretty simple, actually. PSScriptAnalyzer uses the [_Abstract Syntax Tree (AST)_][3] to parse a script file and check for certain patterns depending on the rule. AST has been introduced in PowerShell 3.0.

If you want to find out how to extend PSScriptAnalyzer with your custom rules or read a bit more about it, check the Github repository ([here][4]) or watch a recording of a PowerShell Summit Europe session ([here][5]).

#### Add it to your CI build

You don’t want to build/package code that isn’t up to anyone’s standards, right? There are two ways of achieving a nice code quality in your environment. One is fully-automated, the other one is half-automated.

I use TeamCity CI in my lab to do Continuous Integration builds every time I commit code to some of my repositories. However, I don’t want to have anything built or even uploaded to the Internet if there’s something wrong with my code. That is why I have added this little snippet of code as one of the very first steps into my CI pipeline:

```powershell
try {
   Import-Module -Name PSScriptAnalyzer -ErrorAction Stop
}
catch {
    Write-Error -Message $_
    exit 1
}


try {
     $rules = Get-ScriptAnalyzerRule -Severity Warning,Error -ErrorAction Stop
     $results = Invoke-ScriptAnalyzer -Path "C:\Program Files\WindowsPowerShell\Modules\$Module" -IncludeRule $rules.RuleName -Recurse -ErrorAction Stop
     $results
}
catch {
     Write-Error -Message $_
     exit 1
}
if ($results.Count -gt 0) {
     Write-Host "Analysis of your code threw $($results.Count) warnings or errors. Please go back and check your code."
     exit 1
}
else {
     Write-Host 'Awesome code! No issues found!'
}
```

This code lets the build fail if there are any issues found with it. Very handy, right? This way every commit to this repository’s branch is automatically checked for issues and if any are found, the developer is informed straight away.

But, what if you don’t even want to start building every time, but want to catch issues even earlier on?

#### Git Pre-Commit hooks

Git is AWESOME! (Fullstop!)

Alright, thing is, you want to know about an issue as early as possible. Why even let your bad code get into source control and check it there if you can get that done earlier?

Git has a lot of functionality a lot of people don’t use, like Git hooks.

After creating a local git repository with either _git init_ or _git clone_ have you ever checked that _.&#46;git_ directory?

![](/images/advtools14.png)

That directory has a subdirectory with all possible hooks, even with samples. And they are all bash shell scripts. No worries! That’s easy.

Create two new files in there. Pre-commit (no file extension!!!)

```powershell
#!/bin/sh
echo
exec powershell.exe -ExecutionPolicy Bypass -File '.\.git\hooks\pre-commit-hook.ps1'
exit

pre-commit-hook.ps1
Write-Host 'Starting Script Analyzer'
try {
    $results = Invoke-ScriptAnalyzer -Path .
    $results
}
catch {
    Write-Error -Message $_
    exit 1
}

if ($results.Count -gt 0) {
    Write-Error "Analysis of your code threw $($results.Count) warnings or errors. Please go back and check your code."
    exit 1
}
```

Now commit something. It will now check your code for violations of style best practices. Feel free to change this snippet to your liking. If your code is stylish enough, it gets committed, if not, fix it, and come back.

![](/images/advtools15.png)

In the next part of this two-part series I am going to do quite a bit of testing and validation. Stay tuned.

[1]: /2015/11/02/powershell-tools-for-the-advanced-use-cases-part-2/ "PowerShell tools for the advanced use cases – part 2"
[2]: /2015/11/23/powershell-tools-for-the-advanced-use-cases-part-3/
[3]: https://msdn.microsoft.com/en-us/library/system.management.automation.language.ast(v=vs.85).aspx
[4]: https://github.com/PowerShell/PSScriptAnalyzer
[5]: https://www.youtube.com/watch?v=d7wqwgrWxGE