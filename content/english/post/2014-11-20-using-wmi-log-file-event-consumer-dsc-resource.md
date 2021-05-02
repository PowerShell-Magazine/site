---
title: Using WMI Log File Event Consumer DSC resource
author: Ravikanth C
type: post
date: 2014-11-21T01:39:33+00:00
url: /2014/11/20/using-wmi-log-file-event-consumer-dsc-resource/
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

In today&#8217;s article, I will show you how the <a href="http://msdn.microsoft.com/en-us/library/aa394615(v=vs.85).aspx" target="_blank">WMI Log File Event Consumer</a> <a href="https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMILogFileConsumer" target="_blank">DSC resource</a> can be used. I will show you how this specific DSC resource is used and then show you a complete configuration script using this resource to write to a log file every time a removable device is inserted into the system.

### Creating DSC Log File Event Consumer

We can write to text log files in response to an event using the Log File Event Consumer. This has three mandatory properties.

![](/images/logfile1.png)

The _FileName_ property takes the full path to the log file that needs to be created. If the file exists, the contents specified in the _Text_ property will be appended. If the file does not exist, it will be created. The _Name_ property is the unique identity given to the resource instance. In the optional properties, you can use the _MaximumFileSize_ to auto-rotate the log file after a certain size. By default, this is set to 65,535. The final and another optional property is the _IsUnicode_ property. If you want the log file to be encoded in Unicode, you can set it to _True_.

The following configuration script shows how to use this resource.

```
cWMILogFileConsumer LogFileConsumer {
   Name = 'UFDLogFile'
   Filename = 'C:\Logs\Backup.log'
   Text = 'Removable drive with volume name backup is found. Backup will be initiated.'
   Ensure = 'Present'
}
```


In the above configuration script, we are writing to C:\Logs\Backup.log every time we find a removable drive gets inserted with a volume label &#8216;Backup&#8217;.

Coming to the complete configuration script, we have an event filter that triggers every time a removal device with volume name &#8216;Backup&#8217; is attached to the system, the above configuration script that writes to a log file in response to the event, and an event binding that binds the filter and consumer together.


    Configuration BackuptoUFD {
        Import-DscResource -Module cWMIPermanentEvents
        cWMIEventFilter UFDDetection {
           Name = 'UFDFilter'
           Query = "SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE
           TargetInstance ISA 'Win32_Volume' AND
           TargetInstance.Label='Backup' AND
           TargetInstance.DriveType=2"
           EventNamespace = 'root\cimv2'
           Ensure = 'Present'
        }
    
        cWMILogFileConsumer UFDLogFile {
           Name = 'UFDLogFile'
           Filename = 'C:\Logs\Backup.log'
           Text = 'Removable drive with volume name backup is found. Backup will be initiated.'
           Ensure = 'Present'
        }
    
        cWMIEventBinding UFDCommandLineBinding {
           Filter = 'UFDFilter'
           Consumer = 'UFDLogFile'
           ConsumerType = 'LogFile'
           DependsOn = '[WMIEventFilter]UFDDetection','[WMILogFileConsumer]UFDLogFile'
           Ensure = 'Present'
        }
    }
    
    BackuptoUFD
This is it. This brings us to the end of this article. In the next article in this series, we will look at the <a href="https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventLogConsumer" target="_blank">NT Event Log consumer DSC resource</a>. Stay tuned!