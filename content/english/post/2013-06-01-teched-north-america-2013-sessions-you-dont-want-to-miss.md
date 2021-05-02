---
title: TechEd North America 2013 sessions you don’t want to miss
author: Aleksandar Nikolic
type: post
date: 2013-06-01T13:43:46+00:00
url: /2013/06/01/teched-north-america-2013-sessions-you-dont-want-to-miss/
categories:
  - News
tags:
  - conferences

---
We are all eager to see what Microsoft will announce at the [TechEd North America 2013][1] conference that starts on Monday. You've probably heard about changes coming in [Windows 8.1][2], but what about changes in Windows Server and System Center? What sessions should you attend, if you are interested in the future of Windows Server, System Center, and Windows Azure? The Content Catalog gives us a clue&#8211;there are a lot of sessions with &#8220;to be Announced&#8221; in the title. Windows PowerShell can help us get a list of those super secretive sessions and a little bit more details about them.

We'll use the web cmdlets&#8211;_Invoke-RestMethod_ and _Invoke-WebRequest_&#8211;introduced in Windows PowerShell 3.0. The starting point is the TechEd NA 2013&#8217;s RSS feed. It doesn&#8217;t expose the info about dates and rooms, so we need to use the _Invoke-WebRequest_ cmdlet and filtering to scrape the needed information. Then, we'll create a custom object combining the properties we want and pipe the output to a grid view using the _Out-GridView_ cmdlet where we can further sort and filter the sessions.

**Note**: This tip requires PowerShell 3.0 or above.

    $irm = Invoke-RestMethod -Uri 'http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/RSS'
    $irm | where {$_.Title -match 'to be Announced'} | foreach {
        $iwr = Invoke-WebRequest -Uri $_.link
        $date = $iwr.AllElements | where {$_.tagName -eq 'li' -and $_.class -eq 'date'}
        $room = $iwr.AllElements | where {$_.tagName -eq 'li' -and $_.class -eq 'room'}
    
        [PSCustomObject]@{
          Date = $date.outertext -replace '^date: '
          Room = $room.innerText
          Presenter = $_.Creator
          Link = $_.Link
          Category =  $_.Category
        }
    } | Out-GridView

This is how the output looks like:

![](/images/TechEdNA2013_sessions.png)

**Update:** All titles for &#8220;to be Announced&#8221; sessions have been revealed:

![](/images/TechEdNA2013_sessions_updated.png)

[1]: http://northamerica.msteched.com/
[2]: http://blogs.windows.com/windows/b/bloggingwindows/archive/2013/05/30/continuing-the-windows-8-vision-with-windows-8-1.aspx