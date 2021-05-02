---
title: Troubleshooting WMI standard event consumer issues
author: Ravikanth C
type: post
date: 2014-11-13T17:00:11+00:00
url: /2014/11/13/troubleshooting-wmi-standard-event-consumer-issues/
featured_image: /wp-content/uploads/2011/09/PSMagLogoWhite.jpg
views:
  - 6831
post_views_count:
  - 1742
categories:
  - WMI
tags:
  - WMI

---
While testing the DSC resource module I created for managing the [WMI permanent event filters, consumers, and bindings][1], I came across a bunch of issues around the consumers not doing the expected job. While there are different reasons why these consumers might fail, the way I could troubleshoot those issues is more important.

To give you an idea about WMI permanent event subscriptions, it involves three basic steps. First one is creating an event filter that specifies the type and characteristics of event to monitor. The second step is to determine the right consumer for the type of action that needs to be performed. There are five standard WMI consumers out of the box. And, the final step is to bind the event filter and the chosen consumer instance together.

Before we go into the details, let us take a brief look at the standard consumers.

The following table lists the standard consumers. Source: <http://msdn.microsoft.com/en-us/library/aa392395(v=vs.85).aspx
    

| Standard consumer                                            | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [**ActiveScriptEventConsumer**](http://msdn.microsoft.com/en-us/library/aa384749(v=vs.85).aspx) | Executes a script when it receives an event notification. For more information, see [Running a Script Based on an Event](http://msdn.microsoft.com/en-us/library/aa393250(v=vs.85).aspx). |
| [**LogFileEventConsumer**](http://msdn.microsoft.com/en-us/library/aa392277(v=vs.85).aspx) | Writes customized strings to a text log file when it receives an event notification. For more information, see [Writing to a Log File Based on an Event](http://msdn.microsoft.com/en-us/library/aa394615(v=vs.85).aspx). |
| [**NTEventLogEventConsumer**](http://msdn.microsoft.com/en-us/library/aa392715(v=vs.85).aspx) | Logs a specific message to the Application event log. For more information, see [Logging to NT Event Log Based on an Event](http://msdn.microsoft.com/en-us/library/aa392282(v=vs.85).aspx). |
| [**SMTPEventConsumer**](http://msdn.microsoft.com/en-us/library/aa393629(v=vs.85).aspx) | Sends an email message by using SMTP each time an event is delivered to it. For more information, see [Sending Email Based on an Event](http://msdn.microsoft.com/en-us/library/aa393279(v=vs.85).aspx). |
| [**CommandLineEventConsumer**](http://msdn.microsoft.com/en-us/library/aa389231(v=vs.85).aspx) | Launches a process when an event is delivered to a local system. The executable file must be in a secure location, or secured with a strong access control list (ACL) to prevent an unauthorized user from replacing your executable with a different executable. For more information, see [Running a Program from the Command Line Based on an Event](http://msdn.microsoft.com/en-us/library/aa393249(v=vs.85).aspx). |

The third and final step I mentioned above is essential and without this, the chosen consumer instance will never be used for the configured event filter.

Now, with the DSC resource module I published, achieving the three steps I mentioned above is very easy. In fact, if there is an issue creating one of these instances (filter, consumer, and binding), you will see it right away. However, the actual functionality of the binding cannot be seen until the event really triggers and a consumer bound to that event filter tries to get into action. So, how do you troubleshoot issues with the consumer that is not working as expected?

The WMI infrastructure provides a way to do this. When a consumer fails, WMI creates an instance of [__ConsumerFailureEvent][2] which contains the _ErrorCode_, _ErrorDescription_, and _ErrorObject_ properties. So, if we subscribe to this event class, we can gather the details around a consumer failure through this.

For the demonstration purpose, I will use the SMTP consumer to send an email whenever a new process gets created. But, before that, I will create a temporary event registration for the ___ConsumerFailureEvent_ class. This is available in all WMI namespaces but since the consumers are created in the root\subscription namespace, we will use the same for this event class.

```
Register-CimIndicationEvent -Namespace root/subscription -ClassName __ConsumerFailureEvent -Action { $Global:FailedEvent = $Event } | Out-Null
```

With the above temporary event registration, whenever a consumer fails, we get that instance into the _$FailedEvent_ variable in the global scope that can be further examined. For the consumer part, like I mentioned, I will use SMTP consumer and bind it to a process creation event. Here is the DSC configuration script for that. You will need the [custom DSC resources I published on Github][3].

```
Configuration PermEventDemo3 {
	Import-DscResource -Module cWMIPermanentEvents
	Node Localhost {
            cWMIEventFilter ProcessEventFilter {
                Name = 'ProcessEventFilter'
                Query = "SELECT * FROM __InstanceCreationEvent WITHIN 5 WHERE TargetInstance ISA 'Win32_Process'"
                Ensure = 'Present'
            }

            cWMISMTPConsumer ProcessSMTPConsumer {
                Name = 'ProcessSMTP'
                Message = 'New Process Created with name %TargetInstance.Name%'
                Subject = 'New Process Created'
                SMTPServer = 'smtp.google.com'
                ToLine = 'ToUser@SomeDomain.com'
                FromLine = 'FromUser@AnotherDomain.com'
                Ensure = 'Present'
            }

            cWMIEventBinding ProcessEventLogBinder {
                Filter = 'ProcessEventFilter'
                Consumer = 'ProcessSMTP'
                Ensure = 'Present'
                ConsumerType = 'SMTP'
                DependsOn = '[WMIEventFilter]ProcessEventFilter','[WMISMTPConsumer]ProcessSMTPConsumer'
            }
}
}

PermEventDemo3
```

![](/images/dsccim.png)

In the above configuration, I created an event filter for any process creation, SMTP consumer instance for sending an email to a specific address and specified the relevant properties such as SMTP server address and so on. After that, the event binding DSC resource is used to bind these two together. The SMTP server I specified in the configuration does not exist (the right one is smtp.gmail.com). So, it is expected to fail.

Let us see what the ___ConsumerFailureEvent_ tells us once it gets triggered.

![](/images/failed.png)

We see that the ErrorObject property is available and has an instance of ___ExtendedStatus_. We also see that the _IntendedConsumer_ property is set to an instance of the configured SMTP consumer class. So, accessing _ErrorObject_ property should tell us more.

![](/images/failed2.png)

This is it. There is no such host called smtp.google.com and we see the correct reason in the error object. This method can be used for any consumer that fails.

[1]: /2014/11/12/dsc-resource-for-managing-wmi-permanent-event-filters-consumers-and-bindings/
[2]: http://msdn.microsoft.com/en-us/library/aa394633%28v=vs.85%29.aspx
[3]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents