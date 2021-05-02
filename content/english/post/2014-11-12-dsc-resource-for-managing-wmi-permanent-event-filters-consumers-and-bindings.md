---
title: DSC resource for managing WMI permanent event filters, consumers, and bindings
author: Ravikanth C
type: post
date: 2014-11-12T17:00:40+00:00
url: /2014/11/12/dsc-resource-for-managing-wmi-permanent-event-filters-consumers-and-bindings/
categories:
  - PowerShell DSC
  - WMI
tags:
  - PowerShell DSC
  - WMI
---
I use WMI/CIM event functionality often in my orchestration scripts and [WMI permanent events][1] play a big role there. You can learn more about WMI permanent events by reading my book on [WMI Query Language][2]. Fellow PowerShell MVP, [Trevor Sullivan][3], created a PowerShell module called PowerEvents that can be used to create permanent WMI filters, consumers, and bind them together. You can use that as well if DSC is not your cup of coffee.

But, being a [DSC fanatic][4], I want to always manage my system configuration using DSC. So, I ended up creating a DSC resource module for managing WMI event filters, standard consumers, and bindings between them. This module supports all five standard WMI consumers &#8211; Script, Commandline, SMTP, Log File, and Event Log.

You can grab this module from my Github repo at <https://github.com/rchaganti/DSCResources>.

I have created seven DSC resources for managing WMI permanent event subscriptions.

  * [WMIEventFilter][5]
  * [WMIEventBinding][6]
  * [WMIActiveScriptConsumer][7]
  * [WMICommandLineConsumer][8]
  * [WMIEventLogConsumer][9]
  * [WMILogFileConsumer][10]
  * [WMISMTPConsumer][11]

| **DSC resource Name**   | **Description**                                              |
| ----------------------- | ------------------------------------------------------------ |
| WMIEventFilter          | Creates an Event filter on the target system                 |
| WMIEventBinding         | Creates a binding between Event filter and Event consumer    |
| WMIActiveScriptConsumer | Creates an instance of script event consumer that can be used to trigger VBScript files or script fragments in response to an event |
| WMICommandLineConsumer  | Creates an instance of commandline consumer that can be used to execute a commandline application in response to an event |
| WMIEventLogConsumer     | Creates an instance of NTEventLog consumer that can be used to log event entries in the application log in response to an event |
| WMILogFileConsumer      | Creates an instance of LogFile consumer that can be used to write messages to a text-based log file in response to an event |
| WMISMTPConsumer         | Creates an instance of SMTP consumer that can be used to send emails in response to an event |

While it is possible to combine all consumers into a single resource, I chose to separate them for ease of authoring as well as usage. The Event Filter and Event Binding do not provide any functionality on their own. They have to be always used along with a standard consumer. The following example shows a basic example of using one of the standard consumer DSC resource with an event filter and binding.

```
Configuration PermEventDemo2 {
	Import-DscResource -Module cWMIPermanentEvents
	Node Localhost {
		cWMIEventFilter ProcessEventFilter {
            Name = 'ProcessEventFilter'
            Query = "SELECT * FROM __InstanceCreationEvent WITHIN 5 WHERE TargetInstance ISA 'Win32_Process'"
            Ensure = 'Present'
		}

        cWMIEventLogConsumer ProcessEventLog {
            Name = 'ProcessEventLog'
            EventID = 10011
            Category = 0
            EventType = 'Error'
            SourceName = 'WSH'
            InsertionStringTemplates = 'New Process Created: %TargetInstance.Name%'
            Ensure = 'Present'
        }

        cWMIEventBinding ProcessEventLogBinder {
            Filter = 'ProcessEventFilter'
            Consumer = 'ProcessEventLog'
            Ensure = 'Present'
            ConsumerType = 'EventLog'
            DependsOn = '[WMIEventFilter]ProcessEventFilter','[WMIEventLogConsumer]ProcessEventLog'
        }
	}
}

PermEventDemo2
```

As you see, using a declarative way to specify the configuration is much easier and clearer. Go ahead and explore the module. This is still work in progress and I will add localization support and help content very soon.

I will also write detailed examples and walk-throughs for each resource in this module in the upcoming posts.

[1]: http://msdn.microsoft.com/en-us/library/aa393014(v=vs.85).aspx
[2]: http://www.ravichaganti.com/blog/ebook-wmi-query-language-via-powershell/
[3]: http://trevorsullivan.net/
[4]: http://www.amazon.com/Windows-PowerShell-Desired-Configuration-Revealed/dp/1484200179/ref=sr_1_1?ie=UTF8&qid=1415785652&sr=8-1&keywords=windows+powershell+desired+state+configuration+revealed
[5]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventFilter
[6]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventBinding
[7]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIActiveScriptConsumer
[8]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMICommandLineConsumer
[9]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventLogConsumer
[10]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMILogFileConsumer
[11]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMISMTPConsumer