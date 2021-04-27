---
title: '#PSTip Getting SharePoint 2010 Health Analyzer alerts report by email'
author: Shay Levy
type: post
date: 2013-05-30T18:00:00+00:00
url: /2013/05/30/pstip-getting-sharepoint-2010-health-analyzer-alerts-report-by-email/
categories:
  - SharePoint
  - Tips and Tricks
tags:
  - Tips and Tricks
  - SharePoint

---
SharePoint 2010 administrators are probably familiar with the following screenshot&#8211;you open the Central Administration page and you&#8217;re presented with a red Health Analyzer alert.

![](/images/sphealth.png)

Clicking the &#8216;View these issues&#8217; link takes you to a page that lists all items that needs an attention.

![](/images/sphealth1.png)

Checking the health alerts page each day can be a daunting task and you might also forget to do so. To avoid that, and to enable multiple team members to be aware of the alerts, you can  send the alerts by email. The Health list view (All Reports) is configured to list all items with a Severity not equal to Success (4).

![](/images/sphealth2.png)

Using the following code, you can read all items, and generate an email that you can send to your team members. Put it in a daily scheduled task on the SharePoint server and you&#8217;re good to go.

```
if ($PSVersionTable) {$Host.Runspace.ThreadOptions = 'ReuseThread'}
Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue

# get the health reports list
$ReportsList = [Microsoft.SharePoint.Administration.Health.SPHealthReportsList]::Local
$FormUrl = '{0}{1}?id=' -f $ReportsList.ParentWeb.Url, $ReportsList.Forms.List.DefaultDisplayFormUrl

$body = $ReportsList.Items | Where-Object {$_['Severity'] -ne '4 - Success'} | ForEach-Object {
    New-Object PSObject -Property @{
        Url = "&lt;a href='$FormUrl$($_.ID)'&gt;$($_['Title'])&lt;/a&gt;"
        Severity = $_['Severity']
        Category = $_['Category']
        Explanation = $_['Explanation']
        Modified = $_['Modified']
        FailingServers = $_['Failing Servers']
        FailingServices = $_['Failing Services']
        Remedy = $_['Remedy']
    }
} | ConvertTo-Html | Out-String

# creating clickable HTML links
$body = $body -replace '&lt;','&lt;' -replace '&gt;','&gt;' -replace '&quot;','"'
$params = @{
	To = 'you@domain.com','manager@domain.com'
	From = 'SPHealth@domain.com'
	Subject = 'Daily Health Analyzer report'
	SmtpServer = 'smtp1'
	Body = $body
	BodyAsHtml = $true
}
Send-MailMessage @params
```

This is how it looks in Outlook (partial view).

![](/images/sphealth3.png)