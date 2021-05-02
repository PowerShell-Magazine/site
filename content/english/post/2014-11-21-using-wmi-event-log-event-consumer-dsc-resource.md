---
title: Using WMI Event Log Event Consumer DSC resource
author: Ravikanth C
type: post
date: 2014-11-21T17:00:47+00:00
url: /2014/11/21/using-wmi-event-log-event-consumer-dsc-resource/
categories:
  - WMI
  - PowerShell DSC
tags:
  - WMI
  - PowerShell DSC

---
If you have not seen the earlier articles in this series, I had written about:

  * <a href="/2014/11/12/dsc-resource-for-managing-wmi-permanent-event-filters-consumers-and-bindings/" target="_blank">DSC resource for managing WMI permanent event filters, consumers, and bindings</a>
  * <a href="/2014/11/13/troubleshooting-wmi-standard-event-consumer-issues/" target="_blank">Troubleshooting WMI standard event consumer issues</a>
  * <a href="/2014/11/18/using-the-wmi-commandline-event-consumer-dsc-resource/" target="_blank">Using the WMI Commandline Event Consumer DSC resource</a>
  * <a href="/2014/11/19/using-wmi-active-script-event-consumer-dsc-resource/" target="_blank">Using WMI Active Script Event Consumer DSC Resource</a>
  * [Using WMI Log File Event Consumer DSC resource][1]

In today&#8217;s article, I will show you how the [WMI Event Log Event Consumer][2] [DSC resource][3] can be used. I will show you how this specific DSC resource is used and then show you a complete configuration script using this resource to write an entry in the application event log every time a removable device is inserted into the system.

### Creating WMI NT Event Log Event Consumer

The WMI Event Log Consumer DSC resource has only two mandatory properties. The _Name_ property uniquely identifies the consumer instance and the _EventID_ is the numerical ID assigned to identify the event log entry. However, it does not make sense to log an event entry without the message text and event source and so on. The other properties of this DSC resource provide a way to specify this.

![](/images/EventLog.png)

Let us see the configuration script that shows an example of using this DSC resource.

<pre class="brush: powershell; title: ; notranslate" title="">cWMIEventLogConsumer UFDEventLog {
   Name = 'UFDEventLog'
   EventID = 10011
   Category = 0
   EventType = 'information'
   SourceName = 'WMI'
   InsertionStringTemplates = 'A new UFD drive with volume name Backup is found'
   Ensure = 'Present'
}
</pre>

In this example, we are writing a log entry with an event message text indicating that a new UFD device with volume name &#8220;Backup&#8221; is attached to the system. The _InsertionStringTemplates_ property is used to specify the message text in the event log entry. This property takes an array of strings. The number of array elements is controlled by the _NumberOfInsertionStrings_ property. By default, this is set to 1 and therefore not shown in the above configuration script. The following complete configuration script shows the event filter, consumer, and the binding that writes an event log entry in response to a _Win32_VolumeChangeEvent_.


    Configuration BackuptoUFD {
        Import-DscResource -Module cWMIPermanentEvents
        cWMIEventFilter UFDDetection {
           Name = 'UFDFilter'
           Query = "SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE
           TargetInstance ISA 'Win32_Volume' AND
           TargetInstance.Label='Backup'"
           EventNamespace = 'root\cimv2'
           Ensure = 'Present'
        }
    
        cWMIEventLogConsumer UFDEventLog {
           Name = 'UFDEventLog'
           EventID = 10011
           Category = 0
           EventType = 'information'
           SourceName = 'WMI'
           InsertionStringTemplates = 'A new UFD drive with volume name Backup is found'
           Ensure = 'Present'
        }
    
        cWMIEventBinding UFDCommandLineBinding {
           Filter = 'UFDFilter'
           Consumer = 'UFDEventLog'
           ConsumerType = 'EventLog'
           DependsOn = '[WMIEventFilter]UFDDetection','[WMIEventLogConsumer]UFDEventLog'
           Ensure = 'Present'
        }
    }
    
    BackuptoUFD
This completes the overview of using WMI Event Log consumer DSC resource. In the next article in this series, we will look at the final [WMI SMTP standard consumer][4] for sending emails in response to an event.

[1]: /2014/11/20/using-wmi-log-file-event-consumer-dsc-resource/
[2]: http://msdn.microsoft.com/en-us/library/aa392282(v=vs.85).aspx
[3]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventLogConsumer
[4]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMISMTPConsumer