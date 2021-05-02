---
title: 'Claus Nielsen’s Favorite PowerShell Tips & Tricks'
author: Claus Nielsen
type: post
date: 2012-06-05T18:00:56+00:00
url: /2012/06/05/claus-nielsens-favorite-powershell-tips-tricks/
views:
  - 18567
post_views_count:
  - 1616
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I use PowerShell practically every day, and I would like to share a few tips and tricks that I use quite often.

First a little bit about myself, I work as a System Administrator for a small financial institution, which has about 200 servers and 250 workstations. I am principally responsible for the Microsoft infrastructure Windows, Active Directly, DNS, Citrix XenAPP, and VMware. Fortunately, most of these products already have good PowerShell integration, so that makes my life a lot easier 🙂

### Test-Online

I very often need to query a bunch of machines to gather some sort of information, or change a setting on them. One issue with querying a lot of machines using WMI for instance, is that if the machine is offline, it may take a while for the request to time out, prolonging the overall execution time of the script.

Because I do this quite often, I wrote a small function that I added to my profile—the script takes either a list of computers, a collection of Quest Active Roles Computer objects or Microsoft AD computer objects, it then tries to ping them, and depending on the result, the objects will have a property added which states if the computer is “pingable” or not.

```
function Test-Online {
	[CmdletBinding()]
	param(  
    	[Parameter(Mandatory=$True,ValueFromPipeline=$True)]
   	 	$ComputerName
	)

	Process {
		Switch ($ComputerName.GetType().FullName) {
   			"System.String" {$CompName = $ComputerName}
    		"Quest.ActiveRoles.ArsPowerShellSnapIn.Data.ArsComputerObject" {$CompName = 		$ComputerName.DnsName}
            "Microsoft.ActiveDirectory.Management.ADComputer" {$CompName = $ComputerName.DNSHostName}
    		default {$CompName = "Input Type Not Matched"}
	}	

	Write-verbose "Server: $CompName"

        If(Test-Connection -Count 3 -ComputerName $CompName -TimeToLive 5 -AsJob | Wait-Job | Receive-Job | Where-Object { $_.StatusCode -eq 0 } ) {
        Add-Member -InputObject $_ -MemberType NoteProperty -Name OnlineStatus -Value $true
        Return $_
        }
        Else {
            Add-Member -InputObject $_ -MemberType NoteProperty -Name OnlineStatus -Value $false
            Return $_
        }
	}
}
```

With the above script, I can test a single computer or a collection of computers if they are online, and since I am using jobs I can test multiple computers at a time, thereby greatly reducing the time it takes.  I add a NoteProperty  called “OnlineStatus” , which is either $true or $false, I can then query this property to check if a computer is online or not, before I do a WMI query against the machine.

So, let’s say I want to test computer Server1 and Server2:

<pre class="brush: powershell; title: ; notranslate" title="">"Server1","Server2" | Test-Online</pre>

If I want to use the Quest AD cmdlets I can simply do:

<pre class="brush: powershell; title: ; notranslate" title="">Get-QADComputer | Test-Online | Select-Object Name, OnlineStatus</pre>

That will run through all computers in AD, and check if they are online, and output their name and “OnlineStatus”.

<pre class="brush: powershell; title: ; notranslate" title="">Show-ControlPanelItem</pre>

Another command I have only started using recently is the Show-ControlPanelItem cmdlet (works only in v3).

This allows you to start control panel applets from PowerShell without having to click the Start menu and choose “Control Panel”, or that I have to remember the *.cpl names, which for some applets are pretty gnarly.

So if I want Internet Explorer settings, I used to run inetcpl.cpl. Now, I can do scp \*internet\*  (scp is an alias I created for Show-ControlPanelItem, and \*internet\* is much easier to remember than inetcpl.cpl.

One thing to be aware of is that if you do Show-ControlPanelItem \*i\*, it will open all Control Panel applets containing the letter “I”.

You can also use Get-ControlPanelItem to get a list of all available Control Panel applets.

#### Get-Folder

Sometimes when looking for a folder, it is easier to use the graphical interface than traversing directories using cd and dir. A long time ago, I wrote a function to get a graphical folder browser using the Shell.Application COM object.

<pre class="brush: powershell; title: ; notranslate" title="">Function Get-Folder {
	$obj = New-Object -ComObject Shell.Application
	$bf = $obj.BrowseForFolder(0,"Choose Folder",0,"")
	$bf.Self.Path
}</pre>

By using the technique that the command within parenthesis gets evaluated first, we are now able to do something like:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem (Get-Folder)</pre>

The above command will open a GUI which you can use to select a file. The Get-Folder is evaluated first, since it is enclosed in (), and then the folder you choose in the GUI is passed to Get-ChildItem which will list its content.