---
title: Using WMI SMTP Event Consumer DSC resource
author: Ravikanth C
type: post
date: 2014-11-24T17:00:41+00:00
url: /2014/11/24/using-wmi-smtp-event-consumer-dsc-resource/
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
  * [Using WMI Log File Event Consumer DSC resource](/2014/11/20/using-wmi-log-file-event-consumer-dsc-resource/)
  * [Using WMI Event Log Event Consumer DSC resource](/2014/11/21/using-wmi-event-log-event-consumer-dsc-resource/)

In today&#8217;s article, I will show you how the [WMI SMTP event consumer][3] [DSC resource][4] can be used. I will show you how this specific DSC resource is used and then show you a complete configuration script using this resource to send an email every time a removable device is inserted into the system.

### Creating WMI SMTP Event Consumer

For obvious reasons, you need to specify the _ToLine_, _FromLine_, _SMTPServer_ properties of the DSC resource. The _Name_ property, as you might have guessed by now, uniquely identifies the resource instance. There are several optional properties. The _Message_ property can be used to attach the body of the email and _Subject_ property specifies the email subject. The _CCLine_ and _BCCLine_ properties are self-explained.

![](/images/cmtp.png)

Here is the configuration script that is used to create an SMTP consumer instance.

<pre class="brush: powershell; title: ; notranslate" title="">cWMISMTPConsumer UFDSMTP {
   Name = 'UFDSMTP'
   Message = 'UFD drive with volume name Backup is attached.'
   Subject = 'UFD Detection'
   SMTPServer = 'smtp1.mymailserver.com'
   ToLine = 'ToUser@SomeDomain.com'
   FromLine = 'FromUser@AnotherDomain.com'
   Ensure = 'Present'
}
</pre>

Make a note that there is no method to authenticate to the SMTP server. So, if the server you specified in the configuration requires authentication, the consumer action will fail. You can use the method I <a href="/2014/11/13/troubleshooting-wmi-standard-event-consumer-issues/" target="_blank">had explained in an earlier post</a> to detect any failures in consumer actions.

The following complete configuration script helps us detect a volume change event and then respond to that using the SMTP event consumer by sending out an email to a specified email address.


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
    
        cWMISMTPConsumer UFDSMTP {
           Name = 'UFDSMTP'
           Message = 'UFD drive with volume name Backup is attached.'
           Subject = 'UFD Detection'
           SMTPServer = 'smtp1.mymailserver.com'
           ToLine = 'ToUser@SomeDomain.com'
           FromLine = 'FromUser@AnotherDomain.com'
           Ensure = 'Present'
        }
    
        cWMIEventBinding UFDCommandLineBinding {
           Filter = 'UFDFilter'
           Consumer = 'UFDSMTP'
           ConsumerType = 'SMTP'
           DependsOn = '[WMIEventFilter]UFDDetection','[WMISMTPConsumer]UFDSMTP'
           Ensure = 'Present'
        }
    }
    BackuptoUFD
This is the final post in this series of DSC resources for managing WMI permanent event filters, consumers, and bindings. I have a TODO list which I will share as a readme in the <a href="https://github.com/rchaganti/DSCResources/" target="_blank">Github repo</a>. Feel free to contribute code or report any bugs.

[3]: http://msdn.microsoft.com/en-us/library/aa393629(v=vs.85).aspx
[4]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMISMTPConsumer