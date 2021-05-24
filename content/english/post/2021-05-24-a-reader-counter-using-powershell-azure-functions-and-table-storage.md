---
title: A reader counter using PowerShell, Azure functions, and table storage
author: Ravikanth C
images:
  - '/images/readercounter.png'
type: epic
date: 2021-05-24
url: /2021/05/24/a-reader-counter-using-powershell-azure-functions-and-table-storage/
categories:
- Azure Functions
- Azure Tables
- Azure
tags:
- Azure Functions
- Azure Tables
- Azure
---

We have recently moved PowerShell Magazine from [WordPress to Hugo static pages](https://powershellmagazine.com/2021/05/03/the-all-new-powershell-magazine-is-here-and-more-to-come/). One downside of this move is that we miss a few features that come by default in WordPress. One such feature is the page views or the reader counter. A statically generated page is just HTML, some JavaScript. and no backend whatsoever. So, gathering the views or reader count should happen a user visits a page. I looked around a bit but thought it will be a fun project to write something on my own. So, I started experimenting with my personal blog first and this article is about how I built a reader counter using PowerShell, Azure functions, Azure table storage, and a little bit of JavaScript. Read on.

### Thought Process

For the page views or count of readers who visited a specific page, you need to get a hook when the page gets accessed. This is what happens in a WordPress plugin. Plugin code gets executed when the page is accessed and the related count for the page URL in the backend gets incremented. For a static this is bit of a challenge since you don't have any backend as such. When you have to create a page view counter or a reader counter, you *need to do something when a page is accessed* and then *get something back into the page* to display as the reader count. When I started thinking about an implementation for this, Azure functions came to my mind. Azure functions have a HTTPS endpoint that can be triggered which in turn triggers a PowerShell script that does some magic of storing/retrieving the reader counter values for a given URL.

To make this happen, I needed a few Azure resources.

- Azure Storage account with a Table
- An Azure function app (PowerShell)

There is already ton of information around [creating and publishing an Azure function](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-powershell?tabs=azure-cli%2Cbrowser) and this article won't repeat that. Also, there is a great article on working with [Azure Table storage using PowerShell](https://docs.microsoft.com/en-us/azure/storage/tables/table-storage-how-to-use-powershell). I strongly recommend that you go through these articles before you attempt re-creating what is described in this article.

#### PowerShell function

For enabling a reader counter, I created and published a PowerShell function following the [steps outlined in the article I already mentioned](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-powershell?tabs=azure-cli%2Cbrowser). Here are some changes I made to the function files before publishing.

For accessing AzTable module cmdlets, you need to create a dependency on the module. This is done by updating the requirements.psd1.

```powershell
# This file enables modules to be automatically managed by the Functions service.
# See https://aka.ms/functionsmanageddependency for additional information.
#
@{
    'Az' = '5.*'
    'AzTable' = '2.*'
}
```

I also modified the function.json to allow POST method only.

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    }
  ]
}
```

Finally, here is what the run.ps1 looks like.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Connect to the Storage table
$statacc = Get-AzStorageAccount -ResourceGroupName 'statsrg' -Name 'statsacc'
$statsContext = $statacc.Context
$cloudTable = (Get-AzStorageTable –Name 'statstable' –Context $statsContext).CloudTable

# Interact with query parameters or the body of the request.
$url = $Request.Query.url
$url = $url.Split('/')[2]

if ($url) {
    [string]$filter = `
        [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("RowKey",`
        [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,$url)

    $article = Get-AzTableRow `
        -table $cloudTable `
        -customFilter $filter

    if ($null -ne $article)
    {
        $oldviewCount = $article.views

        # Change the entity.
        $article.views = $postviews = $oldviewCount + 1

        # To commit the change, pipe the updated record into the update cmdlet.
        $article | Update-AzTableRow -table $cloudTable
    }
    else
    {
        Add-AzTableRow `
            -table $cloudTable `
            -partitionKey 'rchaganti' `
            -rowKey ($url) -property @{"views"=1}
        
        $postViews = 1
    }
}
else
{
    $postViews = null
}

$HttpResponse = [HttpResponseContext]@{ 
 StatusCode = 'OK'
 Body = @{"views" = $postViews } | ConvertTo-Json 
 ContentType = "Application/json" 
}

Push-OutputBinding -Name Response -Value $HttpResponse
```

Since this script accesses Azure resource information and updates Azure resources, you need to assign a Managed System Identity (MSI) and assign the appropriate role. To make sure I can update the Azure Table storage, I chose to assign storage account contributor role. There may be a more restrictive role that would work but I did not focus much on figuring out that yet. Maybe I should. 

This script expects a query string parameter called Url and that will be the page a reader is visiting. You will have to replace the values of resource group name, storage account name, and table storage name in the above script. My blog uses permalinks of the format /blog/link-to-an-article/. Line 13 in the above example is specific to my blog URLs as I want to store the final leaf object as the RowKey in the Azure Table. Line 16 creates a filter to check if the RowKey already exists in the table storage. If it exists, I retrieve the views from the row, then increment it, and finally update the row again.

If the RowKey does not exist, I add a new row (line 36) and set views to 1. Finally, I package the response as JSON and push it to client. Here is the output from a test run.

{{< figure src="/images/funcExec.png" >}} {{< load-photoswipe >}}

This is simple and took me less than an hour to figure this out. The fun part was trying to integrate this into the static site and took me more than half day for the lack of any web development experience. You may skip the next part if you use a different static page generator platform.

#### Hugo integration

I love [Hugo](https://gohugo.io/) for all the flexibility it provides in extending the functionality easily. To add the reader counter, I first created a JavaScript that I can use to reach the Azure function endpoint when the article page loads.

```javascript
function getViews(url) {
    var request = new XMLHttpRequest()
    var fullUrl = 'https://sitestats.azurewebsites.net/api/HttpTrigger1?code=wKQ==&url=' + url
    request.open('POST', fullUrl, true)

    request.onload = function (e) {
        if (request.readyState === 4) {
            if (request.status === 200) {
                var data = JSON.parse(this.response)
                document.getElementById("views").innerHTML = data.views
                //console.log(data.views);
            } else {
                console.error(request.statusText);
            }
        }
    };

    request.onerror = function (e) {
        console.error(request.statusText);
    };

    request.send(null);
}
```

This code is self-explanatory (now that I figured out! :)) and it invokes the Azure function URL along with the value of Url (of the article page) passed to this function. 

For this JavaScript to be able to invoke the Azure function endpoint, CORS must be enabled and the domain where you plan to use this JavaScript must be allowed.

{{< figure src="/images/cors.png" >}}

This JavaScript function uses an async HTTP request because you don't want this call to block the page from loading. However, this has a funny side effect. You won't see the reader count appear as soon as page loads but there will be small lag as the request has to make a round trip all the way to azure functions and get the views value. See this in action here.

{{< figure src="/images/blog.gif" >}}

This JavaScript enabled me to get the views value for a specific page. Based on the theme that you use, the following procedure may be different but briefly, here is what I changed.

I updated single.html to add the reader count in the post front matter.

```html
        <div class="post-meta">
          <span class="reading-time">
            <i class="fas fa-eye"></i>
            <strong id="views"></strong>
            {{ i18n "readers" }}
          </span>
        </div>
```

And, added a condition to the baseof.html to invoke the JavaScript function on page load.

```html
{{ if not .Site.IsServer }}
	<body onload="getViews( {{ .RelPermalink }} )" class="{{ $csClass }}{{ if .Site.Params.rtl }} rtl{{ end }}">
{{ else }}
	<body class="{{ $csClass }}{{ if .Site.Params.rtl }} rtl{{ end }}"> 
{{ end }}
```

The condition in this snippet ensures that I invoke the JavaScript function only when the page is accessed from the real site and the local development environment.

Overall, it was fun weekend project to make this work. The application may be very trivial but the learning through this whole process was worth the time I spent on this.
