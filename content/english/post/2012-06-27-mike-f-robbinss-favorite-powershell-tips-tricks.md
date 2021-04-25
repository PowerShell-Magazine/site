---
title: 'Mike F Robbins’s Favorite PowerShell Tips & Tricks'
author: Mike Robbins
type: post
date: 2012-06-27T18:00:04+00:00
url: /2012/06/27/mike-f-robbinss-favorite-powershell-tips-tricks/
views:
  - 30304
post_views_count:
  - 2720
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Instead of an introduction about what I was asked to do and how I went about it, I decided to cut to the chase and give you a bonus tip on how to learn a PowerShell cmdlet a day. I saw this tip on twitter earlier this year and I honestly don’t remember who twitted it or I would give them credit: Want to learn PowerShell? Start every morning with &#8220;Get-Command | Get-Random | Get-Help -Full&#8221;.

### Don’t hard code any type of Format or Output cmdlet into your scripts

Whether you’re writing a long complicated PowerShell script or a one liner that you’re saving as a script, design your code for reusability. One of the most common things I see is hardcoding Format-List, Format-Table, Out-File, and Export-CSV cmdlets into PowerShell scripts which limits how the output of the script can be changed on the fly without having to manually modify it each time. There’s almost no reason to hard code these cmdlets into a script since it can be piped to any of these which accomplishes the same task while maximizing the scripts versatility. The only exception would be if you are automating a task that’s calling a PowerShell script and it will only ever be used for this one thing. I’m going to use a very basic example because it’s not about how complicated the script is, but it’s about the concept of writing versatile scripts that are reusable.

This first script is named Get-CPUProcessTime1.ps1 and while it appears to produce the same output on the screen as the second script, the type of object it produces is different which limits its reusability.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process |
Format-Table @{label='CPU Time';Expression={$_.Cpu}},
   @{Label='Process Name'; Expression ={$_.ProcessName}}
</pre>

When trying to pipe this script to the Out-GridView cmdlet, an error is received because of the type of object that format cmdlets produce.

![](/images/MikeTips111.png)

You also notice when trying to pipe it to the Sort-Object cmdlet, an error is produced:

![](/images/MikeTips12.png)

The version of this script shown below is named Get-CPUProcessTime2.ps1 and it uses the Select-Object cmdlet for the hash table instead of using the Format-Table cmdlet as in the previous script. Whether you’re choosing specific parameters or using a hash table, I recommend using the Select-Object cmdlet for accomplishing these tasks instead of a Format cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process |
Select-Object @{Label='CPU Time';Expression ={$_.Cpu}},
   @{Label='Process Name';Expression ={$_.ProcessName}}
</pre>

This version of the script can be piped to the Out-GridView cmdlet without error:

![](/images/MikeTips1.png)

It can also be piped to the Sort-Object cmdlet without error:

![](/images/MikeTips2.png)

The third example is the same as the second except the Out-File cmdlet is hardcoded in the script:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process |
Select-Object @{Label='CPU Time';Expression ={$_.Cpu}},
   @{Label='Process Name';e={$_.ProcessName}} |
Out-File d:\tmp\test.txt
</pre>

Guess what happens when this version of the script is piped to the Out-GridView cmdlet?

![](/images/MikeTips3.png)

The file is created as specified in the script and the pipe to the Out-GridView cmdlet is ignored. That’s because the Out-File cmdlet doesn’t produce an object so you’re piping nothing to the Out-GridView cmdlet. These same results are produced when using the Export-CSV cmdlet instead of Out-File.

![](/images/MikeTips4.png)

Let’s say I wanted to use the Format-Table cmdlet for the –AutoSize parameter to make the output look a little nicer on the screen, there’s still no reason to hard code that into the script since it can easily be piped to Format-Table without losing its versatility:

![](/images/MikeTips5.png)

PowerShell cmdlets are like Lego blocks, you can build anything you want and for the most part you’re only limited by your imagination, but hard coding Format, Out, and Export cmdlets in your scripts is like super gluing your Lego blocks together (they’re no longer reusable). This applies even if you’re only writing one liners that are being saved as scripts.

### You have to learn how to use a hammer before you can build a house

This means you need to learn the three fundamental PowerShell cmdlets that are the building blocks to all others. These are Get-Help, Get-Command, and Get-Member. These three cmdlets are the key to understanding PowerShell. The first tip leads into this one since I referenced objects in the first tip and piped one of my scripts to the Get-Member cmdlet to determine what type of object it produced.

The Get-CPUProcessTime.ps1 script that I created produces a bunch of Format objects, here’s the first one which is a FormatStartData object. These types of objects aren’t usable by most other cmdlets with the Out-File and other Out-* cmdlets being the only exceptions that I’m aware of.

![](/images/MikeTips6.png)

The second script I created (Get-CPUPRocessTime2.ps1) produces a process object and is a normal object type that can be used by many other cmdlets such as Sort-Object which was shown in the first tip.

![](/images/MikeTips7.png)

There’s lots of great information out there on these three cmdlets and each of them could each take the entirety of this article. For more information on Get-Help and Get-Command see my [blog article][1] on them.

### Write efficient scripts whether they’re being run locally or if they’re using PowerShell remoting

Filter Left and Filter Remote. While I think this tip is a no-brainer or a given, I still continue to see scripts posted on the Internet by other people who consider themselves to be PowerShell enthusiast that aren’t using these best practices. Here’s an example that came from a recent blog on the Internet. I’ve changed the module that’s being used so it can’t be tracked back to the person who wrote it:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command|where{$_.modulename -eq "activedirectory"}|%{$_.name}
</pre>

![](/images/MikeTips8.png)

The syntax is hard to read since it contains almost no spacing. This was an exact copy of what I found on the blog article I referenced and not the way I would write this script. It returns a list of all the cmdlets in the ActiveDirectory PowerShell module. Wrapping it in the Measure-Command cmdlet shows that it takes 886 milliseconds to return the results:

<pre class="brush: powershell; title: ; notranslate" title="">Measure-Command {
	Get-Command|where{$_.modulename -eq "activedirectory"}|%{$_.name}
}
</pre>

![](/images/MikeTips9.png)

Here’s how to filter left and efficiently accomplish the same task:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Command -Module ActiveDirectory | Select-Object -ExpandProperty Name
</pre>

![](/images/MikeTips10.png)

Wrapping this version in the Measure-Command cmdlet shows it completed in just 17 milliseconds:

<pre class="brush: powershell; title: ; notranslate" title="">Measure-Command {
	Get-Command -Module ActiveDirectory | Select-Object -ExpandProperty Name
}
</pre>

![](/images/MikeTips13.png)

The first script takes 5112 percent more time to accomplish the same task since it doesn’t filter left and it is what I would call an inefficient script.

If you’re using PowerShell remoting, you want to filter as much as possible on the remote host (inside your script block) since that will take advantage of distributed computing and it will also generate less network traffic than bringing every result back to your computer only to filter them down to a subset at that point. As the Microsoft Scripting Guy, Ed Wilson said this year during his PowerShell session at SQL Saturday 111 in Atlanta, don’t buy the whole box of Whitman’s samplers if all you like is chocolate covered peanuts.

[1]: http://mikefrobbins.com/2012/03/29/how-do-i-get-started-with-powershell/