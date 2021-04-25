---
title: Getting PowerShelled in 10 Minutes
author: Tobias Weltner
type: post
date: 2012-05-02T18:00:50+00:00
url: /2012/05/02/getting-powershelled-in-10-minutes/
views:
  - 11652
post_views_count:
  - 3154
categories:
  - How To
  - Learning PowerShell
tags:
  - How To
  - Learning PowerShell

---
Windows PowerShell is an extremely powerful scripting language, and when you first put your hands on PowerShell, you may get &#8211; frustrated! There are so many tiny but relevant details: curly braces, brackets, commas and what not. If you get one wrong, all fails. Bummer. That&#8217;s why I decided to view PowerShell like a video game, [poker][1] online games or more like those games I play at [casinodames.com][2] that make me feel so satisfied, [click to view][3] more about casinos online. You wouldn&#8217;t start your favorite ego shooter in expert level either and get blasted away all the time by the boss monster. Instead, conquer PowerShell step by step. Let&#8217;s start with PowerShell game level 1. You&#8217;ll be amazed what this level will enable you to do!

### Mission Briefing

Here are your mission details for today: You need to know what a &#8220;cmdlet&#8221; (say: &#8220;commandlet&#8221;) is, and you need to know how to submit additional information, the &#8220;parameters&#8221;, to a cmdlet. So let&#8217;s start our fighter pilot briefing by launching Windows PowerShell.

### Launching PowerShell

To launch PowerShell, open up the &#8220;Run&#8221; dialog, for example by pressing the keys WIN+R, enter &#8220;powershell.exe&#8221;, and press the Enter key. If all works out, an ugly black window opens (which is good because coincidentally this ugly window hosts the most modern scripting language). Or, Windows complains about a missing command: you may still be running Windows XP or Server 2003 which do not come with PowerShell built-in, so you need to find the optional (and free) download and install it before you can play with PowerShell. Visit your favorite Internet search engine and search for &#8220;KB968930 Windows XP&#8221; for example (or visit http://support.microsoft.com/kb/968929 directly if you can remember that URL).

Next take a quick look inside that black window. Does it read &#8220;Windows PowerShell Copyright (C) 2009&#8221;? If it has a 2006 copyright, then you installed the outdated PowerShell version 1.0. Upgrade to version 2.0 right away! Use the tip above for searching and finding the correct download.

### Configuring the Console

Once you launched PowerShell, you&#8217;ll see a button for your PowerShell window in your taskbar. On Windows 7, right-click that button and pin it to the taskbar, then close the window. Open it again by clicking the pinned PowerShell button. Magic, magic: this time, the PowerShell console paints its background blue. You just opened a preconfigured PowerShell. Aside from the pleasant color, the screen buffer now is 3000 lines high (rather than the default 300 lines) which is nice because PowerShell cmdlets tend to spit out a lot of data, and you don&#8217;t want the information to scroll out of your buffer. When you right-click your pinned taskbar button, a jump list opens and gives you quick access to the other PowerShell components, like a help file and the simple PowerShell editor &#8220;Windows PowerShell ISE&#8221;.

On Pre-Windows 7, to open the preconfigured blue PowerShell window, visit the Accessories program group and open PowerShell there. You can also always click the icon on the left side of the opened PowerShell console window title bar, and then choose &#8220;Properties&#8221; to manually adjust all the console settings. Make sure you set the screen buffer to at least 3000 lines, and pick the colors and font you like best.

### Playing With Cmdlets

To make PowerShell do something, you need to know what your weapons are. Most of what PowerShell can do is encapsulated in a &#8220;cmdlet&#8221;, a basic command that is responsible for one specific area or topic.

To find the right weapon for a mission, each cmdlet has a formal name. It consists of a verb (what it does, weapon category) and a noun (what it acts upon, its target range). The mother of all cmdlets is called &#8220;Get-Command&#8221;: it &#8220;gets&#8221; you all other &#8220;commands&#8221;.

```powershell
PS> Get-Command
```

As it turns out, Get-Command not only returns cmdlets but also other command types that are beyond the current video game level.

### Mission 1: Getting Cmdlet List

Our first fighter mission: &#8220;Obtain a list of all cmdlets available!&#8221;

Get-Command is what we need but it is returning too much information. Here is some PowerShell wisdom: &#8220;The only way to fine-tune a cmdlet is by providing extra parameters&#8221;. What could be the parameter to make Get-Command list only commands of type &#8220;cmdlet&#8221;?

There is yet another old fighter pilots&#8217; trick: when you look at the output, you&#8217;ll notice that it is organized in columns, and each column has a column header. If you want to filter the result by the content of any of these columns, look for a parameter that is called like the column header!

Uhm, that&#8217;s right: there _is_ a column called &#8220;CommandType&#8221; that tells the type of command. Is there a parameter -CommandType, too? Try this:

<pre>PS&gt; Get-Command -Comm[Press TAB]</pre>

Pressing TAB invokes autocompletion, and PowerShell completes the parameter name, so it really exists! Submit the information that we want to filter the result:

<pre>PS&gt; Get-Command -CommandType cmdlet</pre>

Heck, that worked! A parameter here is a key-value pair. The parameter name starts with a &#8220;-&#8220;, and after the parameter name, you add the argument that you want to bind to that parameter. To discover all parameters a cmdlet supports, either use autocompletion:

<pre>PS&gt; Get-Command -[Press TAB now repeatedly]</pre>

Each time you press TAB, PowerShell suggests another parameter to you. If you typed TAB too quickly and passed the parameter you were after, press SHIFT+TAB and try not to break your fingers while you do.

Or, use Get-Help:

<pre>PS&gt; Get-Help -Name Get-Command -Parameter *</pre>

This gets you a formal list of all parameters, what they do and which data types they support.

### Mission 2: Find The Best Weapon For The Job!

Here&#8217;s your second mission: &#8220;Try and find a cmdlet that can read event log entries!&#8221; That&#8217;s an important mission because finding cmdlets is something you&#8217;ll be doing often in PowerShell.

Get-Command can list all cmdlets, but it can do even more. It supports additional parameters that help you search for and find cmdlets in no time. As you have seen, each cmdlet name has a verb and a noun. Coincidentally, Get-Command has two parameters called -Verb and -Noun that you can use to search for cmdlets. This gets you all &#8220;Get&#8221;-cmdlets:

<pre>PS&gt; Get-Command -Verb Get</pre>

Use wildcards to narrow down the results even more. Try and search for the keyword &#8220;event&#8221;:

<pre>PS&gt; Get-Command -Verb Get -Noun *Event*</pre>

This time, you got back only four cmdlets, and Get-EventLog looks promising. If in doubt, you could use Get-Help to check out what the cmdlets really do:

<pre>PS&gt; Get-Help -Name Get-EventLog</pre>

### Mission 3: Getting Error Information from a Whacky Server

Here&#8217;s our third mission: &#8220;Grab the latest 20 error events from the system event log, and once that worked, do it for an alien remote system.&#8221;

You know already the name of the cmdlet you need: Get-EventLog! Try and run Get-EventLog to see what happens next. On a side note, cmdlets starting with &#8220;Get&#8221; are always safe because they just read and never change things. You shouldn&#8217;t dare to &#8220;try out and see what happens&#8221; cmdlets that start with &#8220;Stop&#8221;, &#8220;Remove&#8221;, &#8220;Clear&#8221; or similar verbs. It may be brave, but that could easily turn out to be a career-limiting move. Remember, in a video game you may have 5 credits, in production systems that&#8217;s not always so. Stick to &#8220;Get&#8221;-cmdlets for now, and carefully read the information Get-Help provides for any other cmdlet before you use it.

When you run Get-EventLog, PowerShell prompts you for a &#8220;LogName&#8221;. That tells you that &#8220;LogName&#8221; is a mandatory parameter. Without it, the cmdlet can&#8217;t do anything for you. So either submit the argument for &#8220;LogName&#8221;, for example by entering &#8220;System&#8221;, or press CTRL+C and start over again:

<pre>PS&gt; Get-EventLog -LogName System</pre>

Success! Except, you only wanted the error events, and Get-EventLog dumped the entire log. You know how to fine-tune the cmdlet, right? Examine the column headers, identify the one you want to use for filtering, and add more parameters to Get-EventLog. The column listing the types of event entry is called &#8220;EntryType&#8221;, so use the parameter -EntryType:

<pre>PS&gt; Get-EventLog -LogName System -EntryType Error</pre>

Mission almost accomplished. Now let&#8217;s limit this to the latest 20 error events. Use autocompletion or Get-Help to examine the additional parameters Get-EventLog supports:

<pre>PS&gt; Get-EventLog -LogName System -EntryType Error  -Newest 20</pre>

Likewise, to address one or more remote systems, add the -computername parameter:

<pre>PS&gt; Get-EventLog -LogName System -EntryType Error -Newest 20 -ComputerName server1

PS&gt; $servers = 'server1', 'server2', 'server3'
PS&gt; Get-EventLog -LogName System -EntryType Error -Newest 20 -ComputerName $servers</pre>

As you see, many parameters allow multiple arguments (arrays), so you can query more than one server remotely (provided you have appropriate access permissions). Or, you can list errors _and_ warnings:

<pre>PS&gt; Get-EventLog -LogName System -EntryType Error, Warning  -Newest 20</pre>

### Mission 4: Getting Hotfix Information

Here is mission 4: &#8220;Get a list of all security hotfixes on your machine! Then, try the same with a remote alien machine!&#8221;

First, find the cmdlet for the job. It starts with &#8220;Get&#8221;. What would be the noun part of its name? Search for _fix_:

<pre>PS&gt; Get-Command -Verb Get -Noun *fix*</pre>

Right on: Get-Hotfix! Add a &#8220;-&#8221; and press TAB to view the available parameters. Can you come up with a solution? This one works on English and German systems:

<pre>PS&gt; Get-Hotfix -Description *security*, *sicher*</pre>

### Mission 5: Restarting a Service

Our next mission is going to change something: &#8220;Restart the Spooler Service!&#8221;

This time, we don&#8217;t know the verb, but we do know the noun: &#8220;Service&#8221;! Let&#8217;s try:

PS> Get-Command -Noun Service

Whoa, PowerShell lists a family of cmdlets that all deal with services! It takes no time to identify &#8220;Restart-Service&#8221; as the one we need. Listing cmdlet families is a great way to extend your scope. After you completed this mission, try the same with Get-Eventlog: Get-Command -Noun EventLog.

<pre>PS&gt; Restart-Service -Name Spooler</pre>

This time, you may get red error messages. The error message tells you what&#8217;s wrong: if the Spooler service has dependant services, you need to use the -Force parameter. Also, make sure you run PowerShell with full privileges. Restarting a service is nothing a regular user can do. To open a fully privileged PowerShell, try this:

<pre>PS&gt; Start-Process -Name powershell -Verb RunAs</pre>

In it, try the mission solution:

<pre>PS&gt; Restart-Service -Name Spooler -Force</pre>

Done!

### Mission Debriefing

With only a minimum of PowerShell details, you were able to complete a number of missions, and with this basic knowledge, you can solve many more. Once you know one cmdlet, you know them all. They all work basically the same. Let&#8217;s do a quick debriefing.

You have seen how Get-Command can get you the cmdlet for a job, and how you submit extra information by adding parameters to your call. To get more information about a cmdlet, use Get-Help:

<pre>PS&gt; Get-Help Restart-Service</pre>

Most of us never start script code by scratch. Instead, we search the Internet, find samples and adjust them to our needs. In high school, this was called cheating. In your company, it&#8217;s probably called collaboration. Cmdlets bring tons of sample code that you can use as a starting point for your own code:

<pre>PS&gt; Get-Help Restart-Service -Examples</pre>

To list all parameters a cmdlet supports, use -Parameter:

<pre>PS&gt; Get-Help Restart-Service -parameter *</pre>

If you replace &#8220;Get-Help&#8221; by &#8220;help&#8221;, the information is displayed page by page. You can also redirect it to a file and open the file:

<pre>PS&gt; Get-Help -Name Restart-Service -Parameter * &gt; $env:temp\params.txt</pre>

<pre>PS&gt; Invoke-Item -Path $env:temp\params.txt</pre>

Most parameters consist of a key and a value, but that&#8217;s not always so. The parameter -Force was just a key, and no value was passed.

In PowerShell video game level 1, stick to named and switch parameters only, and ignore positional parameters. To use positional parameters, you need intimate knowledge about a cmdlet and must know at which position the cmdlet expects an argument. They are used by PowerShell Pros to speed up interactive PowerShelling. Compare these three lines. They all do the same but use different ways of shortening commands and parameters:

<pre>PS&gt; Get-Childitem -Path c:\windows -Filter *.exe -Recurse -ErrorAction SilentlyContinue
PS&gt; Get-Childitem  c:\windows *.exe -Recurse -ErrorAction SilentlyContinue
PS&gt; ls c:\windows *.exe -r -ea 0</pre>

For the time being, it is clever to stick to the verbose command syntax we used throughout this article. Have fun playing with PowerShell! Next time around, we&#8217;ll enter PowerShell video game level 2!

[1]: https://www.singapoker.org/
[2]: https://www.casinodames.com/slots
[3]: https://www.canon-ixus95.co.uk