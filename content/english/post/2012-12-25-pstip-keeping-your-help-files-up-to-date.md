---
title: '#PSTip Keeping your help files up to date'
author: Shay Levy
type: post
date: 2012-12-25T19:00:06+00:00
url: /2012/12/25/pstip-keeping-your-help-files-up-to-date/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0.

In PowerShell 3.0 you need to run the _Update-Help_ cmdlet in order to stay up-to-date whenever new help content is available. The problem? You don&#8217;t know when new content is released (unless you visit the [Updatable Help Status Table][1] page periodically).

To ensure that your system has the latest help files, you can register a scheduled job that runs at specific intervals and runs the _Update-Help_ cmdlet. In the following example, _Update-Help_ will execute every day at 05:00 AM.

<pre class="brush: powershell; title: ; notranslate" title="">Register-ScheduledJob -Name UpdateHelp `
		      -ScheduledJobOption @{RunElevated=$true} `
		      -ScriptBlock {Update-Help -Force -Verbose} `
		      -Trigger @{At='5:00 AM';Frequency='Daily'}
</pre>

You can find the newly created task in Task Scheduler UI, on the left pane, under the _Task Scheduler Library\Microsoft\Windows\PowerShell\ScheduledJobs_ task folder.

Once the job has finished running you can get its result by using the job cmdlets:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Job -Name UpdateHelp | Receive-Job -Keep
</pre>

For more information about scheduled jobs, see <a title="about_Scheduled_Jobs" href="http://go.microsoft.com/fwlink/?LinkID=244816" target="_blank">about_Scheduled_Jobs</a>.

[1]: http://go.microsoft.com/fwlink/?LinkID=270007