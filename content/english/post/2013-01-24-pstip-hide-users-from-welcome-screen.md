---
title: '#PSTip Hide users from Welcome Screen'
author: Shay Levy
type: post
date: 2013-01-24T19:00:37+00:00
url: /2013/01/24/pstip-hide-users-from-welcome-screen/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Windows XP and above allow multiple user accounts to be created on the computer. When the computer is in a workgroup mode and there&#8217;s more than one user account defined on that system, Windows will display a Welcome Screen with all available user accounts (as picture icons) , so users can click them to log in into the system.

![](/images/welcome-1024x744.png)

Sometimes however, you will not want to display that list and expose the users on that machine. Using a simple Registry hack you can hide any user account from the Welcome Screen.

Hiding a user is just a matter of adding a new DWord value with the name of the user, User1, and setting its value to &#8216;0&#8217;:

<pre class="brush: powershell; title: ; notranslate" title="">$path = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\SpecialAccounts\UserList'
New-Item $path -Force | New-ItemProperty -Name User1 -Value 0 -PropertyType DWord -Force
</pre>

Next time you visit the Welcome Screen, it will look like:

![](/images/welcome1.png)

To unhide the account, delete the value (or set the value to 1)

<pre class="brush: powershell; title: ; notranslate" title="">Remove-ItemProperty $path -Name User1 -Force
</pre>