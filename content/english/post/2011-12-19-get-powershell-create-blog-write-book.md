---
title: Get-PowerShell | Create-Blog | Write-Book
author: Karl Mitschke
type: post
date: 2011-12-19T19:00:02+00:00
url: /2011/12/19/get-powershell-create-blog-write-book
views:
  - 8649
post_views_count:
  - 1483
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
When I was hired into my current position in July 2007, one of my first tasks was to modify an Active Directory attribute for SharePoint contacts. Specifically, we needed to set the attribute “mapiRecipient” to False. The process that created the contacts left the Boolean attribute in a “Not set” state – that is, neither True nor False.

My supervisor said I could use either VBScript or PowerShell to do this.

As we were planning our transition from Exchange 2000 to Exchange 2007, I decided to use PowerShell.

I had come from a background of writing .Net programs in C# and VB.Net so I was not unfamiliar with the underlying structure of PowerShell objects. I was able to fairly quickly create a script to modify the one custom attribute, which is still in use today – albeit in a slightly modified form.

Having a background in .Net can be both a blessing and a curse, depending on how you look at it. I tend to write scripts with a C# style instead of a “PowerShelly” style.

I quickly realized that there was way more to this PowerShell stuff than the simple scripts I was creating, and started reading blogs by [Shay Levy][1], [Brandon Shell][2], [Marco Shaw][3], [Lee Holmes][4], [Marc van Orsouw][5], as well as the official [Microsoft PowerShell blog][6], and hanging out in the now defunct Microsoft newsgroups (replaced by the [Windows PowerShell][7] forum on TechNet). Pretty soon I started thinking I could actually offer help to people as a way of paying back the community for all the help I’d been provided. Marco in particular started noticing and suggested I create my own blog. Then, less than a year later, Marco asked me if I wanted to work on a book. It took a little over a year, but the “[Windows PowerShell 2.0 Bible][8]” was published in September.

I’m still nowhere near as competent with PowerShell as the people who have unknowingly mentored me, but I have high hopes.

Would you like to share your story? Win one of [TrainSignal’s][9] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][10]” contest.

[1]: http://blogs.microsoft.co.il/blogs/ScriptFanatic/
[2]: http://bsonposh.com/
[3]: http://marcoshaw.blogspot.com/
[4]: http://www.leeholmes.com/blog/
[5]: http://thepowershellguy.com/blogs/posh/default.aspx
[6]: http://blogs.msdn.com/b/powershell/
[7]: http://social.technet.microsoft.com/Forums/en-US/winserverpowershell/threads
[8]: http://www.amazon.com/Windows-PowerShell-2-0-Bible-Thomas/dp/1118021983/ref=sr_1_1?ie=UTF8&qid=1323209869&sr=8-1
[9]: http://www.trainsignal.com/default.aspx
[10]: ../2011/12/12/2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/