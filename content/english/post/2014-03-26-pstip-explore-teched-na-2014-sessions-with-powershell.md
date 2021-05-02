---
title: '#PSTip Explore TechEd NA 2014 sessions with PowerShell'
author: Aleksandar Nikolic
type: post
date: 2014-03-26T18:00:54+00:00
url: /2014/03/26/pstip-explore-teched-na-2014-sessions-with-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

TechEd North America 2014 has a nice <a href="http://tena2014.eventpoint.com/topic/list" title="TechEd North America 2014 Content Catalog" target="_blank">Content Catalog</a> page. It provides a few useful search criteria to filter the content and find the sessions you are mostly interested in. That&#8217;s great, but I wanted to search the content catalog using PowerShell. While we wait for TechEd team to make the content catalog available as an OData web service, we can try to scrap the needed information about sessions from the Content Catalog page.

At first, I&#8217;ve tried to fetch the page with the _Invoke-WebRequest_ cmdlet and use the _ParsedHtml_ property and _GetElementsByTagName()_ method, but that was painfully slow. Then I&#8217;ve remembered that I have friends, <a href="https://twitter.com/TobiasPSP" title="Tobias Weltner" target="_blank">Tobias</a> and <a href="https://twitter.com/nohwnd" title="Jakub Jares" target="_blank">Jakub</a>, who are very good with Regular Expressions. Regular Expressions are an esoteric art and difficult to get right, but, luckily, these two guys know their RegEx.

Another challenge was the way how authors of the Content Catalog page deal with the paging. By default, you will get results in the chunks of 10 per result page. With some help of browser&#8217;s developer tools, Jakub&#8217;s found out we can modify the URL using the &#8220;take&#8221; parameter and ask for more (there are 600+ session, so we used &#8220;take=1000&#8221; to get them all).

At the end, we output a custom PowerShell object with sessions&#8217; details and wrap it as a reusable _Search-TechEdNA2014ContentCatalog_ function:   

    function Search-TechEdNA2014ContentCatalog {
        param
        (
            # by default you'll get all sessions from a content catalog
            [string]$Keyword = ''
        )
    
        $Uri = 'http://tena2014.eventpoint.com/Topic/List?format=html&take=1000&keyword=' + $Keyword
        $Results = Invoke-WebRequest -Uri $Uri
    
        $Results.RawContent -replace "\n"," " -replace "\s+"," " -replace "(?<=\>)\s+" -replace "\s+(?=\<)" -split 'class="topic"' |
         select -skip 1 |
     		foreach {
                $Speaker     = if ( $_ -match 'Speaker\(s\):.*?</div>' ) { $matches[0] -split "," -replace ".*a href[^>].*?>" -replace "</a.*"  | foreach { $_.Trim() } }
                $Title       = if ( $_ -match 'Class="title".*?href.*?>(.*?)<' ) { $Matches[1].Trim() }
                $Track       = if ( $_ -match "Track:.*?>(.*?)<" ) { $Matches[1].Trim() }
                $SessionType = if ( $_ -match "Session Type:.*?>(.*?)<" ) { $Matches[1].Trim() }
                $Date        = if ( $_ -match 'class="session">(.*?)<' ) { $Matches[1].Trim() }
                $Description = if ( $_ -match 'class="description">(.*?)<' ) { $Matches[1].Trim() }
    
                [pscustomobject]@{
                    Date = $Date
                    Track = $Track
                    SessionType = $SessionType
                    Speaker = $speaker
                    Title = $Title
                    Description = $Description
        		}
    		}
    	}
What is the best way to use this function? Pipe its output to a grid view window, do some additional filtering if you like and output the results to the console:

```
Search-TechEdNA2014ContentCatalog -Keyword powershell | Out-GridView -PassThru
```


![](/images/TechEdNA2014_Content_Catalog.png)

Or, export the results to a CSV file and open it in Excel:

```
Search-TechEdNA2014ContentCatalog -Keyword azure |
Out-GridView -PassThru |
Select Date, @{n='Speaker(s)';e={$_.Speaker -join ', '}}, Title |
Export-Csv $env:temp\sessions.csv -NoTypeInformation

Invoke-Item $env:temp\sessions.csv
```


