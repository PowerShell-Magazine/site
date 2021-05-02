---
title: '#PSTip How to automatically dot-source all scripts in a folder'
author: David Moravec
type: post
date: 2012-11-01T18:00:36+00:00
url: /2012/11/01/pstip-how-to-automatically-dot-source-all-scripts-in-a-folder/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
As I mentioned in my previous post, I like to have access to (almost) all my scripts on all machines. That’s why I have **Scripts** folder inside my **Dropbox:\PowerShell\Profile**. Anytime I have some nice script I drop it there and can use it.

<pre class="brush: powershell; title: ; notranslate" title=""># Load all scripts
Get-ChildItem (Join-Path ('Dropbox:\PowerShell\Profile') \Scripts\) | Where `
    { $_.Name -notlike '__*' -and $_.Name -like '*.ps1'} | ForEach `
    { . $_.FullName }
</pre>

I am just running **dir** against Scripts folder and then check if the file has the .ps1 extension, but doesn’t start with “**__”** (double underscore). A reason? If I want to temporarily remove just one script from loading I will just rename it and prefix its name with a “__”. So, from the following list:

```
[23]: dir Dropbox:\PowerShell\Profile\Scripts | Sort Name | Select Name

Name
----
__Set-Prompt.ps1
Credential.ps1
Get-EnumValue.ps1
Get-ExceptionDescription.ps1
Get-Menu.ps1
…
```

The first script is not processed. The other scripts are piped to another command where I dot-source them. I also use auto-loading of my own format files.

<pre class="brush: powershell; title: ; notranslate" title=""># Load format files
Get-ChildItem (Join-Path ('Dropbox:\PowerShell\Profile\Scripts') \Format\) | ForEach `
    { Update-FormatData -AppendPath $_.FullName }
</pre>

I use these format files to correctly format output from Configuration Manager WMI provider for some of my own scripts.