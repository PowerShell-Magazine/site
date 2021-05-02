---
title: '#PSTip Decoding an HTML-encoded string'
author: Ravikanth C
type: post
date: 2014-07-09T18:00:21+00:00
url: /2014/07/09/pstip-decoding-an-html-encoded-string/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In my [previous tip][1], I showed some of my favorite methods to convert date time string to the local time. In the same context of RSS feeds, I sometimes find the description text in the RSS feed is HTML-encoded. One example is the Azure service update feed.

```
[Reflection.Assembly]::LoadWithPartialName('System.Web')

$feed = 'http://azure.microsoft.com/en-us/updates/feed/'
$xml = Invoke-RestMethod -Uri $feed
$xml[1].description
```

Output from the above snippet differs based on what is the first item in the feed. When I want a readable text, this is not very helpful. So, here is one method I use to decode this string into something that is meaningful to me.

$xml | Select Title, @{Name='PublicationDate';Expression={[DateTime]$_.PubDate}}, @{Name='Description';Expression={([System.Web.HttpUtility]::HtmlDecode($_.Description)) -replace "<.*?>",''}}


![](/images/html-1024x189.png)

[1]: /2014/07/08/pstip-covert-to-local-time/