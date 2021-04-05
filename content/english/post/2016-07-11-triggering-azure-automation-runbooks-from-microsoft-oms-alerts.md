---
title: Triggering Azure Automation runbooks from Microsoft OMS Alerts
author: Jan Egil Ring
type: post
date: 2016-07-11T16:00:53+00:00
url: /2016/07/11/triggering-azure-automation-runbooks-from-microsoft-oms-alerts/
ratings_average:
  - 4.67
post_views_count:
  - 4082
categories:
  - Azure Automation
  - Azure
tags:
  - Azure Automation
  - Azure

---
Microsoft Operations Management Suite (OMS) is an IT management solution for the era of the cloud.

OMS extends System Center IT management to the cloud, enabling greater control, visibility and security across your hybrid cloud.

That is Microsoft&#8217;s [own description][1] of what OMS is and why you should use it.

In this article we are going to look at a feature in OMS which makes it possible to automatically trigger Azure Automation runbooks based on alerts.

The heart of OMS is the Log Analytics feature, which makes it possible to interact with real-time and historical machine data. These data can be fed into OMS in several ways, but the most common is via the Microsoft Monitoring Agent. Among the data the agent can upload are Windows Event Logs and Windows Performance Counters, but this is something which can be customized in the Data section of the Settings Dashboard:

![](/images/aaoms1.png)

All data uploaded to OMS can be queried using a very powerful search language, not unlike similar products such as Splunk. When accessing the Log Search feature inside the OMS portal, a number of predefined saved searches can be found:

![](/images/aaoms2.png)

Looking at examples is great for getting started, but you can also look up the [Log Analytics search reference][2] for more details.

Note that searching the OMS logs can also be performed from PowerShell, but that is a topic for a different article.

As an example, let us have a look at the predefined search “All computers with missing updates”. When clicking on that search link we are presented with the following logs and options:

![](/images/aaoms3.png)

The results can be viewed in different ways, such as a list and a table. In this example we can also see a customized view for updates. Similarly, there are other customized views based on the data type. For example, performance data have its own customized views.

There is a number of options on the top meny above the results, such as the option to export results and save queries. The feature we are going to focus on and demonstrate in this article is the Alerts feature in OMS Log Analytics.

Before we click on the Alert button, we are going to change the search query to a custom search:

```sql
Type=Perf (ObjectName:LogicalDisk AND CounterName:"% Free Space" AND InstanceName="C:" AND CounterValue<15 AND TimeGenerated>NOW-1HOUR)
```

As you probably can guess based on the query, the scenario we want to enable an alert for is C: volumes with less than 15% of free space.

You can read more about how Alerts in Log Analytics works in the [documentation][3], but in a nutshell it makes it possible to set up a log search which will trigger an alert based on your own criteria:

![](/images/aaoms4.png)

_Image credit: [Microsoft][3]_

The Alert actions can be one of the following:

  * E-mail – an e-mail with details of the alert will be sent to one or more recipients
  * Webhook – Invoke an external process by triggering a single HTTP POST request. An example of a service which supports webhooks is [Slack][4], where for example can define different channels where alerts is fed into.
  * Runbook – This action will trigger a runbook (PowerShell code) in Azure Automation. Under the hood, this is also a webhook. Using this feature requires that you have added and configured the Automation solution in your OMS workspace.

The runbook action is what we are going to demonstrate in this article.

When our custom search is executed (and optionally saved), we click the Alert button:

![](/images/aaoms5.png)

You are then brought to a page where the alert can be customized:

![](/images/aaoms6.png)

You can customize settings such as severity, time window, and frequency. You can find more information about those in the [documentation][3].

The interesting setting in our demo context is the Actions, and specifically the Runbook action. Select Yes, select the runbook you want the alert to trigger and choose whether to execute the runbook in Azure (a sandbox) or on a Hybrid Worker in your own environment. In our demo scenario we want the remediation runbook to take actions against the computers having low disk space, thus I chose to execute the runbook on a Hybrid Worker which do have access to my local network.

After saving the alert rule, we can see that a webhook has been automatically created on the specified runbook in Azure Automation:

![](/images/aaoms7.png)

Next, we will have a look at the runbook we selected in the new OMS Alert, since there are some important considerations to make during authoring.

The whole runbook is available as a Gist on GitHub, and I am going to explain the important parts of it below the code:

First, do note the following statements regarding the runbook:

  * It is meant for demonstration purposes only, use it at your own risk.
  * Features such as error handling, logging and tests should be added prior to use in a production environment.
  * I would recommend to separate the remediation runbook from the actual runbook doing the remediation in order to keep things clean as well as make testing easier. In practice, this would mean that the Invoke-OMSAlertDiskCleanup runbook would call one or more additional runbooks in order to do actions against the computers the alert is generated for.

Now let us have a closer look at the runbook used in our demo scenario.

The first important step is to define a single parameter called $WebhookData of type [object], as this is the parameter OMS Alerts is hardcoded to send information to. It\`s not required to define this parameter, but if you want to retrieve the context from the alert inside the runbook it is highly recommended. When defined, $WebhookData will contain three properties:

![](/images/aaoms8.png)

_Image credit:_ [_Microsoft_][5]

You can read more about details regarding this in the Azure Automation Webhooks [documentation][5].

The RequestBody properties contains data in JSON format, which we can convert to PowerShell objects and store in a variable:

<pre class="lang:ps decode:true">$RequestBody = ConvertFrom-JSON -InputObject $WebhookData.RequestBody</pre>

**Tip:** _During development of a new runbook for use with OMS Alerts it might be useful to inspect this object interactively. You can temporarily add a line in the runbook to export the object to Clixml:_

```powershell
$RequestBody | Export-Clixml -Path C:\Temp\Invoke-OMSAlertDiskCleanup_RequestBody.xml
```

After the alert has triggered at least once, you should be able to get the XML file from the Hybrid Runbook worker and import it in an interactive PowerShell session for inspection:

![](/images/aaoms9.png)

In the scenario we are working with (OMS Alerts) the RequestBody property will contain a property called SearchResults, where we can find the values from the OMS search that triggered the alarm as we can see above.

For our demo runbook where we are interested in doing actions against the computers with less than 15% of free space on the C: drive, we first need to get the computer names. The same computer can occur multiple times, thus we need to extract the unique names:

```powershell
$Computers = $RequestBody.SearchResults.value | Sort-Object -Property Computer -Unique
```

Now that we have the names of the computers, we are ready to take an action to try to remediate the problem (low disk space).

In this demo we will only perform a single action in a foreach loop&#8211;Invoke the Deployment Image Servicing and Management (DISM.exe) command-line tool on the computers via PowerShell remoting to try to clean up installed updates:

```powershell
$Cleanup = Invoke-Command -ComputerName $Computer -ScriptBlock {dism.exe /online /Cleanup-Image /StartComponentCleanup} -Credential $Credential
```

Hopefully, this can free up some disk space and potentially resolve the problem.

There is a number of additional steps I would consider if doing this in a production environment:

  * Add more actions, such as invoking Disk Cleanup (cleanmgr.exe). This might require different logic based on the operating system version, so I did not spend time implementing it in this demo.
  * Add some logic to time stamp the last time a Windows Update cleanup was attempted in order to not invoke it multiple times in a short period of time.
  * If desired and supported by the operating system, you could also consider to automatically expand the C: drive if the computer is a virtual machine and none of the other remediation actions succeeded.
  * Add functionality to determine whether it is allowed to perform this kind of automated maintenance tasks on the computers, something that might not be allowed on critical production systems.

The demonstration scenario is only one of an unlimited number of possible scenarios for automated actions based on a search criterion in OMS. Another scenario can be: “If updates are missing on a computer, trigger the Windows Update client to download and install updates”.

Hopefully this demonstrated how you can leverage the Alerts feature in Microsoft OMS to automatically trigger runbooks in Azure Automation in order to remediate issues in your environment.

[1]: https://www.microsoft.com/en/server-cloud/operations-management-suite/why-oms.aspx
[2]: https://azure.microsoft.com/en-us/documentation/articles/log-analytics-search-reference/
[3]: https://azure.microsoft.com/en-us/documentation/articles/log-analytics-alerts/
[4]: http://www.slack.com/
[5]: https://azure.microsoft.com/en-us/documentation/articles/automation-webhooks/