---
title: Using WMI Active Script Event Consumer DSC resource
author: Ravikanth C
type: post
date: 2014-11-19T17:00:58+00:00
url: /2014/11/19/using-wmi-active-script-event-consumer-dsc-resource/
categories:
  - WMI
  - PowerShell DSC
tags:
  - WMI
  - PowerShell DSC

---
In my previous article, I talked about [using WMI Commandline Event Consumer DSC resource][1]. In that article, I showed you how we can respond to a flash drive insertion event and create a backup copy of a local folder on the removal device. We already saw how to create an event filter to detect a removable drive insertion event. We will reuse that here. So, let&#8217;s directly go the discussion on how we can use the [cWMI Active Script Event Consumer DSC resource][2].

Before we go ahead, understand that the only scripting engine supported by the Active Script Event Consumer is VBScript. You cannot invoke PowerShell scripts directly. However, you can use a VBScript wrapper to run a PowerShell script.

### Creating Active Script Consumer

The [cWMIActiveScriptConsumer DSC resource][2] has two different ways of specifying the script to execute.

![](/images/activescript.png)

The first method is to specify a _ScriptFileName_ that takes the complete path to a .vbs script. The second method is to use the _ScriptText_ property to provide the VBScript fragment that needs to be executed. The _MaximumQueueSize_ and _ScriptingEngine_ are optional properties.

<pre class="brush: powershell; title: ; notranslate" title="">cWMIActiveScriptConsumer UFDScript {
   Name = 'UFDScript'
   ScriptText = '
      Set objFSO=CreateObject("Scripting.FileSystemObject")
      curDate = Year(Date) & "-" & Month(Date) & "-" & Day(Date)
      objFolder = TargetEvent.TargetInstance.DriveLetter & "\" & curDate
      Set bkFolder = objFSO.CreateFolder(objFolder)
      objFSO.CopyFolder "c:\Scripts", objFolder
      objFile.Close
    '
    Ensure = 'Present'
}
</pre>

In the above configuration script, the _TargetEvent.TargetInstance_ gives us access to the _Win32_Volume_ instance object. Using this, we can retrieve the _DriveLetter_ property for the attached removable disk.

You  can copy the contents of _ScriptText_ property to a .vbs file and use that as the argument for the _ScriptFileName_ property. Understand that the _ScriptText_ and _ScriptFileName_ are mutually exclusive.

### Complete Configuration Script

After the event filter and event consumer instance creation, we need to bind them together. We have already seen an example of using WMIEventBinding DSC resource in an earlier article. So, here is the complete configuration script.

<pre class="brush: powershell; title: ; notranslate" title="">Configuration BackuptoUFD {
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
   cWMIActiveScriptConsumer UFDScript {
       Name = 'UFDScript'
       ScriptText = '
           Set objFSO=CreateObject("Scripting.FileSystemObject")
           curDate = Year(Date) & "-" & Month(Date) & "-" & Day(Date)
           objFolder = TargetEvent.TargetInstance.DriveLetter & "\" & curDate
           Set bkFolder = objFSO.CreateFolder(objFolder)
           objFSO.CopyFolder "c:\Scripts", objFolder
           objFile.Close
       '
       Ensure = 'Present'
   }
   cWMIEventBinding UFDCommandLineBinding {
       Filter = 'UFDFilter'
       Consumer = 'UFDScript'
       ConsumerType = 'Script'
       DependsOn = '[WMIEventFilter]UFDDetection','[WMIActiveScriptConsumer]UFDScript'
       Ensure = 'Present'
   }
}
BackuptoUFD
</pre>

This is it for today. In the next article, we will look at how to use the [LogFile Event Consumer DSC resource][3].

[1]: /2014/11/18/using-the-wmi-commandline-event-consumer-dsc-resource/
[2]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMIActiveScriptConsumer
[3]: https://github.com/rchaganti/DSCResources/tree/master/cWMIPermanentEvents/DSCResources/cWMILogFileConsumer