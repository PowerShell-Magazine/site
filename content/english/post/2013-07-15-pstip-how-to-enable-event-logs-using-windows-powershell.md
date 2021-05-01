---
title: '#PSTip How to enable Event logs using Windows PowerShell'
author: Shay Levy
type: post
date: 2013-07-15T18:00:46+00:00
url: /2013/07/15/pstip-how-to-enable-event-logs-using-windows-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Since Windows Vista, the operating system is shipped with a ton of additional event logs and providers, some of them are present on the system but are not enabled. For example, you want to enable the _Microsoft-Windows-DNS-Client/Operational_ log. Among other options, logs can be enabled or disabled by using the built-in command line utility, _[Wevtutil][1]_, but this is a PowerShell tip so we&#8217;re going to use PowerShell to enable the log file.

Note that in order to enable the log the code must run from an elevated console or you will get a _&#8220;Attempted to perform an unauthorized operation.&#8221;_ error. First let&#8217;s examine the state of the log file.

```
PS> Get-WinEvent -ListLog Microsoft-Windows-DNS-Client/Operational | Format-List *
FileSize                       :
IsLogFull                      :
LastAccessTime                 :
LastWriteTime                  :
OldestRecordNumber             :
RecordCount                    :
LogName                        : Microsoft-Windows-DNS-Client/Operational
LogType                        : Operational
LogIsolation                   : Application
IsEnabled                      : False
IsClassicLog                   : False
SecurityDescriptor             : O:BAG:SYD:(A;;0xf0007;;;SY)(A;;0x7;;;BA)(A;;0x7;;;SO)(A;;0x3;;;IU)(A;;0x3;;;SU)(A;;0x3
                                 ;;;S-1-5-3)(A;;0x3;;;S-1-5-33)(A;;0x1;;;S-1-5-32-573)
LogFilePath                    : %SystemRoot%\System32\Winevt\Logs\Microsoft-Windows-DNS-Client%4Operational.evtx
MaximumSizeInBytes             : 1052672
LogMode                        : Circular
OwningProviderName             : Microsoft-Windows-DNS-Client
ProviderNames                  : {Microsoft-Windows-DNS-Client}
ProviderLevel                  :
ProviderKeywords               :
ProviderBufferSize             : 64
ProviderMinimumNumberOfBuffers : 0
ProviderMaximumNumberOfBuffers : 64
ProviderLatency                : 1000
ProviderControlGuid            :
```

To enable it we create a new [_EventLogConfiguration_][2] object and pass it the name of the log we want to configure. We enable it and save the changes.

```
$logName = 'Microsoft-Windows-DNS-Client/Operational'

$log = New-Object System.Diagnostics.Eventing.Reader.EventLogConfiguration $logName
$log.IsEnabled=$true
$log.SaveChanges()

# check the state again
PS> Get-WinEvent -ListLog Microsoft-Windows-DNS-Client/Operational | Format-List is*

IsLogFull    : False
IsEnabled    : True
IsClassicLog : False
```

Using the same method we can configure many other options of the log file, just take a look at the [EventLogConfiguration Class][3] for a list of configurable properties.

[1]: http://technet.microsoft.com/en-us/library/cc732848(v=ws.10).aspx
[2]: http://msdn.microsoft.com/en-us/library/bb338646.aspx
[3]: http://msdn.microsoft.com/en-us/library/system.diagnostics.eventing.reader.eventlogconfiguration.aspx