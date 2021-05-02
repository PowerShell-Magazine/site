---
title: '#PSTip Get next available drive letter'
author: Shay Levy
type: post
date: 2012-08-15T21:46:45+00:00
url: /2012/08/15/pstip-get-next-available-drive-letter/
views:
  - 10471
post_views_count:
  - 2168
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Mapping drives is a routine task for IT pros—you choose a free drive letter, ranging from A-Z, and then use it to map a drive to a specific path. What if you need to do that in a script? How do you know which letter is available for the new drive?

Here at the magazine we dedicated a [Brain Teaser to find an unused drive letter][1] but we want to tell you of another, new, built-in way to get that information.

Windows 8 and Server 2012 ship with the Storage module which provides PowerShell cmdlets for end to end management of Windows storage. One of the module functions is Get-AvailableDriveLetter.

**Update (8/20/2012)**

In Windows 8 RTM, Get-AvailableDriveLetter is not a part of the Storage module.

However it will be included in a downloadable module at the TechNet Script Center in the near future.

In the meantime, you can put the code in your own Get-AvailableDriveLetter function.

Running the function without any parameters outputs all available drive letters:

```
PS> Get-AvailableDriveLetter

G
H
I
...
X
Y
Z
```

We can also ask for the first letter only:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-AvailableDriveLetter -ReturnFirstLetterOnly
G
</pre>

Finally, since this is a function, we can see how letters are retrieved by taking a look at the function definition:


	PS> Get-Command Get-AvailableDriveLetter | Select-Object  –ExpandProperty Definition
	param(
	    [parameter(Mandatory=$False)]
	    [Switch]
	    $ReturnFirstLetterOnly
	)
	
	$volumeList = Get-Volume
	# Get all available drive letters, and store in a temporary variable.
	$UsedDriveLetters = @(Get-Volume | % { "$([char]$_.DriveLetter)"}) + @(Get-WmiObject -Class Win32_MappedLogicalDisk| %{$([char]$_.DeviceID.Trim(':'))})
	$TempDriveLetters = @(Compare-Object -DifferenceObject $UsedDriveLetters -ReferenceObject $( 67..90 | % { "$([char]$_)" } ) | ? { $_.SideIndicator -eq '&lt;=' } | % { $_.InputObject })
	
	# For completeness, sort the output alphabetically
	$AvailableDriveLetter = ($TempDriveLetters | Sort-Object)
	if ($ReturnFirstLetterOnly -eq $true)
	{
		$TempDriveLetters[0]
	}
	else
	{
		$TempDriveLetters
	}
For a list of the cmdlets contained in the Storage module, see the following topic [on TechNet][2]

[1]: /2012/01/12/find-an-unused-drive-letter/)
[2]: http://technet.microsoft.com/en-us/library/hh848705.aspx