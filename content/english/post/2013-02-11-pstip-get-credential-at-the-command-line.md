---
title: '#PSTip Get-Credential at the command line'
author: Shay Levy
type: post
date: 2013-02-11T19:20:21+00:00
url: /2013/02/11/pstip-get-credential-at-the-command-line/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
PowerShell&#8217;s _Get-Credential_ cmdlet lets us create a secure credential object for a specified user name and password using a UI dialog:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Credential shay
</pre>

![](/images/Get-Credential.png)

There&#8217;s a way of replacing the UI and collect the credentials via the command line. You need to be an administrator to do that and the console must be elevated (e.g &#8220;Run as Admin&#8221;).

The change involves adding a registry value to the HKLM hive:

<pre class="brush: powershell; title: ; notranslate" title="">$key = "HKLM:\SOFTWARE\Microsoft\PowerShell\1\ShellIds"
Set-ItemProperty -Path $key -Name ConsolePrompting -Value $true
</pre>

You add the _ConsolePrompting_ value to the above path and set its data to _$true_. From now on, all _Get-Credential_ calls will look like this:

```
PS> Get-Credential shay

Windows PowerShell Credential Request
Enter your credentials.
Password for user shay: ********
```

To bring back the UI dialog, set the value to _$false_ or remove it all altogether. Note that this trick doesn&#8217;t have any effect in the ISE.