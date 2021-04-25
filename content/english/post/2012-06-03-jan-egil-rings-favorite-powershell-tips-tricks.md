---
title: 'Jan Egil Ring’s Favorite PowerShell Tips & Tricks'
author: Jan Egil Ring
type: post
date: 2012-06-03T18:00:39+00:00
url: /2012/06/03/jan-egil-rings-favorite-powershell-tips-tricks/
views:
  - 16485
post_views_count:
  - 1443
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Disclaimer: This article is written based on the PowerShell 3.0 version available in the Windows 8 Consumer Preview. Features described may change in the final version of the product.

### Feature #1 – Enhancements to the PowerShell ISE

I have always used the PowerShell Integrated Scripting Environment (ISE), but not as my primary script editor due to lack of some essential features, like brace matching. This is a very useful feature when working with scripts that contains several loops.

A major change to the ISE is the combining of the input and output panes which exists in PowerShell ISE 2.0:

![](/images/JanTricks1.png)

In PowerShell ISE 3.0 there are 2 panes:

![](/images/JanTricks2.png)

The default background color is also changed to be the same as in the PowerShell console (powershell.exe), which I think makes it look and feel more like PowerShell for people opening the ISE for the first time.

Of course, there are different opinions about this change, as some people do like the input/output panes in PowerShell ISE 2.0, but for me this is a welcome change.

Another useful feature I want to highlight is the snippets functionality. Imagine that you are in the middle of writing a script, and you cannot remember the syntax for a switch statement. With the new snippets feature you can hit Ctrl + J to launch the snippets menu, and then type “s” to get to the switch statement:

![](/images/JanTricks3.png)

Then hit Enter and you will get a sample switch statement to use as a basis:

![](/images/JanTricks4.png)

There are too many features in the ISE to cover them all in this article, but enhanced and vastly improved ISE is definitely one of my favorite features in PowerShell 3.0.

You can read more about new features in PowerShell ISE in the TechNet article [What&#8217;s new in the Windows PowerShell ISE][1]. I would also recommend you to read the “Windows PowerShell ISE.pdf” document released with the [Windows Management Framework 3.0 &#8211; Community Technology Preview (CTP) #2][2].

### Feature #2 – Default parameter values

Another exciting new feature is the $PSDefaultParameterValues preference variable. (For more information about this feature, see about\_Parameters\_Default_Values.) This is best explained with a few examples. Let us say that you want to use the Send-MailMessage cmdlet:

<pre>Send-MailMessage -From you@domain.com -To itops@domain.com `
       -Subject "Hello world" -SmtpServer smtp.domain.com</pre>

If this is a cmdlet you use a lot you will notice that there are some parameters you must specify which is mostly static, in this example the –From and –SmtpServer parameters.

You can define the default parameter values for these parameters in a special type of hash table like this:

<pre>$PSDefaultParameterValues = @{
   "Send-MailMessage:From" = "you@domain.com"
   "Send-MailMessage:SmtpServer" = "smtp.domain.com"
}</pre>

Now the usage of the Send-MailMessage can be shortened to the following:

<pre>Send-MailMessage -To itops@domain.com -Subject "Hello world"</pre>

Another practical example is the –Credential parameter which can allow you to run all cmdlets that accepts the parameter using the specified alternate credentials:

<pre>$cred = Get-Credential
$PSDefaultParameterValues += @{"*:Credential"=$cred}</pre>

When defining the $PSDefaultParameterValues preference variable in your PowerShell [profile][3], the default values you specify will be available in every PowerShell session you open. And if you would like to temporarily turn off the feature, you can disable it like this:

$PSDefaultParameterValues[&#8220;Disabled&#8221;] = $true

### Feature #3 – Job Scheduling

In previous versions there was no native way in PowerShell to work with scheduled jobs, except from a scheduled tasks module available in the [PowerShell Pack][4] which was part of the Windows 7 Resource Kit. Typically you would create a scheduled task which runs powershell.exe, and pass the –Command or –File parameters as arguments to execute your command or script.

In PowerShell 3.0 there is a new module called PSScheduledJob which have the following cmdlets:

![](/images/JanTricks5.png)

The scheduled jobs created using the module can be found in the Windows Task Scheduler under Task Scheduler Library\Microsoft\Windows\PowerShell\ScheduledJobs.

A sample job:

<pre>$jobtrigger = New-JobTrigger -Once -At (Get-Date).AddMinutes(1)
Register-ScheduledJob -Name "PowerShell Scheduled Job Example" `
 -Trigger $jobtrigger -ScriptBlock {
   Import-Module ActiveDirectory
   $lockedaccounts = Search-ADAccount -LockedOut
   if($lockedaccounts){
      $body = $lockedaccounts | Select-Object Name | ConvertTo-Html | Out-String
      Send-MailMessage -From you@domain.com -To itops@domain.com `
       -Subject "Locked accounts" -Body $body -BodyAsHtml `
       -SmtpServer smtp.domain.com
   }
}</pre>

Note: The sample requires that the Microsoft Active Directory module is available on the computer the job is scheduled to run on.

Here we can see that the job is created in the Windows Task Scheduler:

![](/images/JanTricks6.png)

You can read more about the new job scheduling cmdlets in the [Scheduling Background Jobs in Windows PowerShell 3.0][5] article on the Windows PowerShell Team’s blog.

[1]: http://technet.microsoft.com/en-us/library/hh849181.aspx
[2]: http://www.microsoft.com/en-us/download/details.aspx?id=27548
[3]: http://msdn.microsoft.com/en-us/library/windows/desktop/bb613488(v=vs.85).aspx
[4]: http://archive.msdn.microsoft.com/PowerShellPack
[5]: http://blogs.msdn.com/b/powershell/archive/2012/03/19/scheduling-background-jobs-in-windows-powershell-3-0.aspx