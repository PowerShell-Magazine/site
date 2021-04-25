---
title: '#PSTip Storing of credentials'
author: David Moravec
type: post
date: 2012-10-30T18:00:41+00:00
url: /2012/10/30/pstip-storing-of-credentials/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I frequently use different credentials to connect to some of my servers. Oh man, I wish I have just one account J As Configuration Manager is based on WMI, my most frequently used cmdlet is **Get-WmiObject**. And I use its **Credential** parameter to pass credentials I need for specific server. You can save your credentials to a variable by using the **Get-Credential** cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $cred = Get-Credential domain\makovec
</pre>

But after some time it’s boring to do this every time I start my session. So, I am saving my credentials to a file and load it in $profile. I have three functions for working with credentials.

  1. New-DMCredential – creates a new file with credentials stored inside.
  2. Get-DMCredential – load credentials from a file (and then save it to a variable).
  3. Show-DMCredential – ehh, sometimes I forgot current password (especially after holiday). This one shows me password in clear text.

I got this idea of storing it this way from Lee Holmes’s PowerShell Cookbook. I’ve just written a function around his code. You can see dDM alias used in the following examples. This alias is used as a file name for stored credentials (used as output of New-DMCredential function and input of Get-DMCredential). The name of variable holding credential object is also based on this alias name.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; New-DMCredential -UserName domain\makovec -Alias dDM
</pre>

It creates file with credentials. To be clear – the file doesn’t contain data in a clear text. It’s encrypted using Data Protection API.

I can use this stored credentials to assign them to a variable (this is the line I have in my $profile):

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-DMCredential -UserName domain\makovec -Alias dDM
</pre>

When I want to connect to one of my servers I can use this variable:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Connect-ConfigMgrProvider -ComputerName MyServer -Credential $dDM
</pre>

Or I can see my password when needed:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Show-DMCredential $dDM
Pa$$w0rd
</pre>

Here are the functions I use. I&#8217;ve commented it inline.


	function New-DMCredential
	{
		[CmdletBinding()]
	    param(
	        [Parameter(
	            Mandatory = $true,
	            Position = 0
	        )]
	        [string]$UserName,
	        [Parameter(
	            Mandatory = $true,
	            Position = 1
	        )]
	        [string]$Alias
	    )
	
	    # Where the credentials will be stored
	    $path = 'c:\Scripts\Resources\cred\'
	
	    # get credentials for given username
	    $cred = Get-Credential $UserName
	
	    # and save encrypted text to a file
	    $cred.Password | ConvertFrom-SecureString | Set-Content -Path ("$path\$Alias")
	}
	
	function Get-DMCredential
	{
		[CmdletBinding()]
	    param(
	        [Parameter(
	            Mandatory = $true,
	            Position = 0
	        )]
	        [string]$UserName,
	        [Parameter(
	            Mandatory = $true,
	            Position = 1
	        )]
	        [string]$Alias
	    )
	
	    # where to load credentials from
	    $path = 'c:\Scripts\Resources\cred\'
	
	    # receive cred as a PSCredential object
	    $pwd = Get-Content -Path ("$path\$Alias") | ConvertTo-SecureString
	    $cred = New-Object System.Management.Automation.PSCredential $UserName, $pwd
	
	    # assign a cred to a global variable based on input
	    Invoke-Expression "`$Global:$($Alias) = `$cred"
	    Remove-Variable -Name cred
	    Remove-Variable -Name pwd
	}
	}
	
	function Show-DMCredential
	{
		param($cred)
		# Just to see password in clear text
	  		[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($cred.Password))
	}
I can probably change this script to module and make it a bit more user friendly. I will do that soon. At the moment&#8211;you can do that as your homework :).