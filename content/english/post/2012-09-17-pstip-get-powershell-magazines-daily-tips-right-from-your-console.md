---
title: '#PSTip Get PowerShell Magazine’s daily tips right from your console!'
author: Ravikanth C
type: post
date: 2012-09-17T18:00:25+00:00
url: /2012/09/17/pstip-get-powershell-magazines-daily-tips-right-from-your-console/
views:
  - 6684
post_views_count:
  - 1673
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Our editors at the PowerShell Magazine are doing a great job pushing daily PowerShell tips and tricks to our readers. Now, wouldn&#8217;t it be great if you can get access to these tips right from your console?

Yes, it is easy. In fact, PowerShell 3.0 makes it way too easy. Check this:


	Function Get-PSTip {
		param (
			[switch]$Multiple
		)
	    $url = Invoke-RestMethod -Uri "http://bit.ly/QwjgRd"
	
	    if ($Multiple) {
	        $selectedTips = $url | Select-Object Title, Link | Out-GridView -PassThru
	        if ($selectedTips) {
	            $selectedTips | Foreach-Object {
	                Start-Process $_.Link
	            }
	        }
	    } else {
	        Start-Process $url[0].Link
	    }
	}
This is it. Add this simple code snippet to your PowerShell profile and then every time you run, _Get-PSTip_, you will have the latest tip served right from your console or PowerShell ISE prompt. Also, you can use Get_-PSTip_ with _-Multiple_ parameter to select a tip from the list and open that in browser.