---
title: PowerShell and I
author: Aman Dhally
type: post
date: 2011-12-12T19:00:56+00:00
url: /2011/12/12/powershell-and-i/
views:
  - 5065
post_views_count:
  - 999
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
I will start by saying that a person’s overall experience is no indication of his/her knowledge. I have over 10 years of experience and I spent the first five years supporting desktop systems and applications. Then, I changed my job and went to a company which had lots of enterprise clients and this was in the year of 2005 when VB Scripting was very famous among system administrators. I was told by my seniors that scripting was too complex and difficult to learn. I tried learning VB Script and gave up very soon. My journey of learning and using scripting ended even before starting.

After this, things have changed quite a bit for me. I joined my current company and went to UK for an internal training on IT systems. I met several bright people there who inspired me to get started again in scripting. Especially Ben – our IT Manager – was a great scripter himself. He said, “Aman, PowerShell is very easy and it’s very easy to learn” – maybe he was just comforting me, I thought. He gifted me a book on PowerShell – Windows PowerShell 2.0 for dummies.

Once I came to speed, I quickly started using PowerShell for basis activities including:

  * Using PowerShell as a calculator
  * System ping, reboot, and shutdown
  * Working with files and folders
  * Open applications, etc

As Ben said, PowerShell is straight forward and simple to learn. All I needed was some focus and enthusiasm to learn. My first script was, starting a stopped service but with some simple logic.

```powershell
$a = Get-Service WinRM            
if($a.Status -eq "Stopped")
{
	$a.Start()
}
elseIf($a.Status -eq "Running")
{
	echo $a.Name "is running"
}
```

PowerShell scripting seemed easy to me and I finally wrote a 100-line script and posted it on PowerShell.com. There were 73 downloads and I even got an email from a user of this script and he said that he liked it and thanked me for the script. This was the first kick! I told to myself, yes, I am scripter! And, I was very happy.

The “PowerShell community”, are a group of great IT guys and very nice people! Everyone I met or interacted with – [Ed Wilson][1], [Shay Levy][2], and many more – inspired me a lot. Ed helped me create the New Delhi PowerShell User Group and Shay helps almost every day and answered all my questions!

If I have to describe the PowerShell community in a single phrase, I’d say “they are the best”!

I am currently writing two blogs, one is dedicated to the [New Delhi PowerShell User Group][3]  and another one is on [SCOM][4].

Would you like to share your story? Win one of [TrainSignal’s][5] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][6]” contest.

[1]: http://www.scriptingguys.com/
[2]: http://PowerShay.com
[3]: http://newdelhipowershellusergroup.blogspot.com/
[4]: http://opsmgradmin.blogspot.com
[5]: http://www.trainsignal.com/default.aspx
[6]: ../2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/