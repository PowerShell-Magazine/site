---
title: '#PSTip Create mapped network drive using WScript.Network'
author: Jaap Brasser
type: post
date: 2014-07-21T18:00:44+00:00
url: /2014/07/21/pstip-create-mapped-network-drive-using-wscript-network/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
There are multiple methods of mapping network drives in PowerShell. In the past the net.exe tool was used and since PowerShell 3.0 drives can be persistently mapped using the _New-PSDrive_ cmdlet. There is a third option available using the _MapNetworkDrive_ method of WScript.Network COM object.

An example of how a drive can be mapped can be seen here:

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object -ComObject WScript.Network).MapNetworkDrive('Z:','\\server\folder')
</pre>

This will not map the drive persistently. In other words, the drive will disappear after reboot or when a user logs out. To ensure the drive mapping is persistent a third argument should be added&#8211;a _Boolean_ value set to true:

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object -ComObject WScript.Network).MapNetworkDrive('Z:','\\server\folder',$true)
</pre>

It is also possible to specify a username and password. Unfortunately, both the username and password have to be supplied as plain text. An example of how to map a drive using alternate credentials:

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object -ComObject WScript.Network).MapNetworkDrive('Z:','\\server\folder',$true,'domain\user','password')
</pre>

A drive mapping might already be present using the drive letter that we want to use for the new mapped drive. The _RemoveNetworkDrive_ method of the _WScript.Network_ object can be utilized to remove a mapped network drive:

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object -ComObject WScript.Network).RemoveNetworkDrive('Z:')
</pre>

When there are open connections to the mapped drive, an error will be thrown when executing the previous command. To forcefully disconnect a drive mapping add _$true_ as the second argument.

<pre class="brush: powershell; title: ; notranslate" title="">(New-Object -ComObject WScript.Network).RemoveNetworkDrive('Z:',$true)
</pre>

For more details about these two methods and available arguments see the following articles on MSDN:

MapNetworkDrive: <http://msdn.microsoft.com/en-us/library/8kst88h6(v=vs.84).aspx>

RemoveNetworkDrive: <http://msdn.microsoft.com/en-us/library/d16d7wbf(v=vs.84).aspx>