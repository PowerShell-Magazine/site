---
title: My 10 best practices leading to the Scripting Games 2012 – Part I
author: Ifiok Moses
type: post
date: 2012-03-20T18:00:49+00:00
url: /2012/03/20/my-10-best-practices-leading-to-the-scripting-games-2012-part-i/
views:
  - 32663
post_views_count:
  - 4035
categories:
  - How To
  - Scripting Games
tags:
  - Scripting Games
  - How To

---
I wrote an article earlier in the year about how I got inspired into PowerShell. The enthusiasm has made me consider entering for the scripting games, I even decided to [visit site][1] here to learn even more, if you are interested to learn how to play [sbobet][2] poker online game follow us. At the moment I can’t tell how ready I am but having started reading to prepare I have a few things I have come to understand as best practices towards having a successful participation.

While it is possible for everyone to come out with that code that does something, there are several other factors that make one to stand out from the rest.

I title this “10 best practices for the scripting games” since games are really popular including casino games such as [situs judi slot online][3] which is great for entertainment and is a game you can play online. I understand that there may be a lot more things to be aware of but the point here is not to attempt to highlight everything in PowerShell that needs to be addressed but to summarize my learning points into 10 handy notes while preparing, plus I know you are all trying to hurry so that you can play with [slotsmummy][4].

In this two-part write-up I focus on the following:

  * Annotate your script
  * Accept input by pipeline
  * Use CmdletBinding
  * Use Comment Based Help
  * Keep it simple (KISS)
  * Support the enterprise
  * Manage the errors
  * Use native PowerShell commands
  * Output objects
  * Answer the question

### Annotate your script

One of the major reasons for participating in a competition like this is to be able to produce tools that will be useful at work. This means that either you will be using by yourself or giving out to colleagues to use. The code will always require to be reviewed, either to improve functionality, or to add more. It is important that when you pick up the code to review you know and understand exactly what the code was originally designed to do.

Let your code outlive you. In your organization you will move to other roles and will not be managing codes anymore or out of the company.  If someone, your successor, or colleague, or someone you mentor happens to review that code he should be able to do something with it.

Below is an example of an annotated in code that I have taken particular note of are these few lines taken from [Jeff Hick’s _MorningReport_ script][5].

```powershell
if($computername -eq $env:COMPUTERNAME)
{
   #local computer so no ping test is necessary
   $OK=$True
}
elseIf(($ComputerName -ne $env:COMPUTERNAME) -AND (Test-Connection `
-ComputerName $ComputerName -Quiet -Count 2))
{
   #not local computer and it can be pinged so proceed
   $OK=$True
}

if($OK)
{
   Try {
       $os = Get-WmiObject Win32_OperatingSystem -ComputerName `
             $ComputerName -ErrorAction Stop
       #set a variable to indicate WMI can be reached
       $wmi=$True
(...)
```

The learning point for me here is how every line of code seems to have a statement explaining its purpose. Jeff Hicks has written on several aspect of PowerShell and his blog on jdhitsolutions.com/blog forms a great resource for both experienced PowerShell users, and beginners.

### Accept pipeline input

The Scripting Guy in his last interview with PowerScripting podcast used the analogy of a mechanical tool he buys from a store to use. When you go to the store to buy this tool, you expect it to work straight out for the box. You have the object you want to use it on and you have expected results. It is completely unimportant to you at that moment what the tool is made of. Understandably if you are that savvy you want to know more about the specs, the physical makeup of the device, etc., but how many people really do that? The bottom line is that anyone taking that tool throws in his input and expects an output.

Whenever we use native PowerShell cmdlets we take for granted that these will be able to take in a single input or a collection of inputs from the pipeline, process them, and produce results. And in most cases, they do. As for what the expected results should be, the section on outputting objects discusses this. However it is important to understand that some of the best scripts will sit in the middle while ingesting objects (or stings) from the pipeline and spitting objects as outputs to be used further down the pipeline &#8211; something similar to a factory process.

PowerShell’s advanced function makes provision for this by providing the “accepts input by pipeline” option in the param block. Utilizing this contributes to producing a function that behaves like a native cmdlets.

A good article online is <http://www.windowsitpro.com/print/scripting/Handling-Input-in-PowerShell-Functions> by Bill Stewart.

### Use CmdletBinding Attribute

By definition, CmdletBinding “declares a function that acts similar to a compiled cmdlet”.

One of the greatest attributes of PowerShell is the consistency in structure. It just makes sense that whenever we write our scripts we attempt to make them look and feel and act as close to native cmdlets as possible, as already discussed in the previous section.

CmdletBinding automatically provides the function with parameters like –Verbose, -Debug, -ErrorVariable, -ErrorAction commonly called as common parameters.

Read “[What does PowerShell&#8217;s [CmdletBinding()] Do?][6] “:                [http://www.windowsitpro.com/blog/powershell-with-a-purpose-blog-36/windows-powershell/powershells-[cmdletbinding]-142114][6]  to further understand this.

You may wish to read the online help

  * <http://msdn.microsoft.com/en-us/library/dd347560>
  * <http://technet.microsoft.com/en-us/library/dd315326.aspx>

In addition, including “ShouldProcess” and setting that to $true gives us the ability to include -WhatIf and -Confirm to our script.

### Use comment-based help

There is a difference between the comments you write in the script as annotations and comment-based help used for developing the help text for your script. While the ordinary comments, with‘#’ at the beginning of a new-line is used to help you or any other person reading the code in the future, comment based help extends this to actually providing a help file for your function..

In its most basic form <http://powershell.com/cs/blogs/tips/archive/2009/11/06/using-comment-based-help.aspx> is an excellent place to start.

Jeff Hicks wrote a wonderful article on his blog about a year ago and it can be found here.           <http://jdhitsolutions.com/blog/2011/03/new-comment-help/>

If you are not comfortable with answering a series of read-host questions to build your help file you can use ScriptingGuy’s Add-help function here:                <http://blogs.technet.com/b/heyscriptingguy/archive/2010/09/11/automatically-add-comment-based-help-to-your-powershell-scripts.aspx>

Or if you good at reading, you can as well read through: <http://technet.microsoft.com/en-us/library/dd819489.aspx>

Again including this in you script ensures allows people to use get-help to understand how the script works and how it can be used – further making the script to behave like a native cmdlet.

### Keep it simple (KISS)

The most efficient codes are not necessarily the most complex. On the contrary when the code is simple and straightforward, it is easier to read, execute, and review; it also runs more efficiently, etc.

I came across the following two codes while looking for a simple solution – number of mailboxes per database.

Solution A:

```powershell
Get-Mailbox | Group-Object -Property Database | Select-Object Name, Count
```


Solution B:

```powershell
Get-MailboxDatabase | Select-Object Name, `
@{Name="Count";Expression={(Get-Mailbox -Database $_.Identity | Measure-Object `
).Count}}}
```


(Please note that original code has been modified to provide the desired results.)

These two codes give same results. I decided to pass them through a **Measure-Command** just for the fun of it and the result was interesting:

Solution A came out in 1.75s:

```powershell
PS> Measure-Command {Get-Mailbox | Group-Object -Property Database | `
Select-Object Name, Count}
Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 1
Milliseconds      : 747
Ticks             : 17472628
TotalDays         : 2.02229490740741E-05
TotalHours        : 0.000485350777777778
TotalMinutes      : 0.0291210466666667
TotalSeconds      : 1.7472628
TotalMilliseconds : 1747.2628
```


Solution B took 3.02s

```powershell
PS> Measure-Command {Get-MailboxDatabase | Select-Object Name, `
@{Name="Count";expression={(Get-Mailbox -Database $_.Identity | `
Measure-Object).Count}}}

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 3
Milliseconds      : 20
Ticks             : 30205752
TotalDays         : 3.49603611111111E-05
TotalHours        : 0.000839048666666667
TotalMinutes      : 0.05034292
TotalSeconds      : 3.0205752
TotalMilliseconds : 3020.5752
```

This was carried out in an environment with about 200 mailboxes. I can only imaging the impact of this in an environment of 10,000 mailboxes.

This concludes the first part of this write-up. Please come back for the second and concluding part as we discuss remoting, error handling and objects.

[1]: https://no.slotzo.com/
[2]: https://sbobetasia55.com/
[3]: https://newmacau88.co/
[4]: https://slotsmummy.com/payment-methods/
[5]: http://jdhitsolutions.com/blog/2012/01/the-powershell-morning-report/
[6]: http://www.windowsitpro.com/blog/powershell-with-a-purpose-blog-36/windows-powershell/powershells-%5bcmdletbinding%5d-142114