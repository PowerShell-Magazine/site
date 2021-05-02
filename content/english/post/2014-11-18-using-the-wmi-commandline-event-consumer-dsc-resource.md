---
title: Using the WMI Commandline Event Consumer DSC resource
author: Ravikanth C
type: post
date: 2014-11-18T17:00:03+00:00
url: /2014/11/18/using-the-wmi-commandline-event-consumer-dsc-resource/
featured_image: /wp-content/uploads/2011/09/PSMagLogoWhite.jpg
views:
  - 6841
post_views_count:
  - 1699
categories:
  - PowerShelll DSC
  - WMI
tags:
  - WMI
  - PowerShell DSC

---
In my earlier article, I promised a detailed walk-through of each DSC resource to manage the WMI permanent events. To start that series, I will show how to use the command line Event Consumer DSC resource. The event filter and event binding resources are very straightforward. So, I will talk about these two resources as a part of this article.

For the demo purpose, let&#8217;s look at something more useful than just monitoring process creation or deletion. In this article, we will see an event filter that detects insertion of a USB flash drive and then triggers the command line Event Consumer to perform backup of a local folder to the newly inserted USB flash drive. Also, I want the backup to be performed only when the flash drive&#8217;s volume label is &#8216;Backup&#8217;.

### Creating an Event Filter

For detecting a USB flash drive insertion, we can use the [Win32_VolumeChangeEvent][1] class. The _EventType_ property tells us what type of volume change event occurred. However, the event object does not contain a volume label. So, instead of this event class, I will use an intrinsic event class named _\_InstanceCreationEvent and watch for any new Win32\_Volume instances. Using this, I get the ability to filter the event based on the label of the inserted flash drive. In the below query, we use _DriveType=2_ to ensure that we detect only removable disks.

<pre class="brush: powershell; title: ; notranslate" title="">$Query = "SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE
                   TargetInstance ISA 'Win32_Volume' AND
                   TargetInstance.Label='Backup' AND
                   TargetInstance.DriveType=2"
</pre>

Now that we have the query, let&#8217;s see how an event filter can be created using the c[WMIEventFilter DSC resource][2]. You can get the syntax for this resource by using the _-Syntax_ parameter with the _Get-DscResource_ cmdlet.

![](/images/filter1.png)

As you see here, there are only two mandatory properties&#8211;_Name_ and _Query_. The _Name_ property must be unique on the target system. The _Query_ property takes a WMI query, like the one I showed above, as input. The _EventNamespace_ is set to root\cimv2, by default. You can set it based on where the event gets triggered. The event filter instance gets created in the root\subscription namespace.

The following code snippet shows the configuration script for a resource instance:

<pre class="brush: powershell; title: ; notranslate" title="">cWMIEventFilter UFDDetection {
   Name = 'UFDFilter'
   Query = "SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE
                TargetInstance ISA 'Win32_Volume' AND
                TargetInstance.Label='Backup' AND
                TargetInstance.DriveType=2"
   EventNamespace = 'root\cimv2'
   Ensure = 'Present'
}
</pre>

### Creating an instance of Commandline Consumer

We have an instance of the event filter created for the USB flash drive insertion. What we need next is a standard consumer instance that performs the action we need. In this article, we will look at the c[WMICommandLineConsumer][3].

![](/images/command1.png)

The _Name_ and the _CommandLineTemplate_ properties are the mandatory in the Commandline Consumer DSC resource. If you specify the program name to execute as part of the _CommandLineTemplate_ property, you should avoid specifying that as the _ExecutablePath_ property and vice versa. Before we create the event consumer instance using DSC, we need a batch command file that copies the data to the newly inserted USB flash drive. We will put a simple xcopy command in a .cmd file and store it in C:\Scripts folder.

<pre class="brush: powershell; title: ; notranslate" title="">@ECHO OFF
FOR /f %%a in ('WMIC OS GET LocalDateTime ^| find "."') DO set DTS=%%a
set TODAY=%DTS:~0,4%-%DTS:~4,2%-%DTS:~6,2%
Set BackupFolder=%1\%TODAY%
mkdir %BackupFolder%
xcopy /E /Y C:\DSCDemo\*.* %BackupFolder%
</pre>

If you see the above batch file, I am creating a folder at the destination to represent a date when the backup was taken. Also, I had hard-coded the folder that needs to be copied. C:\DSCDemo is the folder that I will be copying to. If you are wondering how the batch script knows which drive letter to use for the newly inserted removable disk, that will be sent to the batch script as a commandline argument. The %1 in the batch script represents the first argument sent to the script.

With this handy, let us create the Commandline Consumer instance using DSC.

<pre class="brush: powershell; title: ; notranslate" title="">cWMICommandLineConsumer UFDCommandLine {
   Name = 'UFDCommandLineConsumer'
   CommandLineTemplate = 'cmd.exe /c C:\Scripts\Backup.cmd %TargetInstance.DriveLetter%'
   Ensure = 'Present'
}
</pre>

In the configuration script, if you look at the _CommandLineTemplate_ property, I am passing the drive letter of the removable disk as _%TargetInstance.DriveLetter%_.

### Creating a Filter to Consumer Binding

The binding between the event filter and consumer can be created using [c_WMIEventBinding_ DSC resource][4].

![](/images/binding2.png)

This resource takes three mandatory properties. These are _Filter_, _Consumer_, and _ConsumerType_. For the Filter and Consumer properties, you should use the name of the instance you specified in the configuration. In my configuration, it will be _UFDFilter_ and _UFDCommandLineConsumer_. For the _ConsumerType_ property, the valid values are listed in the screenshot above. For this example, we will need to specify _CommandLine_ as the argument for the ConsumerType property. Here is how the configuration script looks:

```
cWMIEventBinding UFDCommandLineBinding {
   Filter = 'UFDFilter'
   Consumer = 'UFDCommandLineConsumer'
   ConsumerType = 'CommandLine'
   DependsOn = '[WMIEventFilter]UFDDetection','[WMICommandLineConsumer]UFDCommandLineConsumer'
   Ensure = 'Present'
}
```


Now that we have the configuration script for all three, it is time to see the complete configuration script for a target system.

```
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

    cWMICommandLineConsumer UFDCommandLine {
       Name = 'UFDCommandLineConsumer'
       CommandLineTemplate = 'cmd.exe /c C:\Scripts\Backup.cmd %TargetInstance.DriveLetter%'
       Ensure = 'Present'
    }

    cWMIEventBinding UFDCommandLineBinding {
       Filter = 'UFDFilter'
       Consumer = 'UFDCommandLineConsumer'
       ConsumerType = 'CommandLine'
       DependsOn = '[WMIEventFilter]UFDDetection','[WMICommandLineConsumer]UFDCommandLine'
       Ensure = 'Present'
    }
}
```

I have not used _Node_ block in the configuration script and therefore this generates the configuration MOF for the local system. After we enact this configuration, every time a removable disk with &#8216;Backup&#8217; volume name is inserted, the backup.cmd script gets triggered and copies contents of C:\DSCDemo to the removable disk.

[1]: http://msdn.microsoft.com/en-us/library/aa394516(v=vs.85).aspx
[2]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventFilter
[3]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMICommandLineConsumer
[4]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIEventBinding