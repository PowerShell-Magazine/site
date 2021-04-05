---
title: Redfish Event Listener in PowerShell
author: Ravikanth C
type: post
date: 2018-11-13T09:00:51+00:00
url: /2018/11/13/redfish-event-listener-in-powershell/
post_views_count:
  - 8051
views:
  - 7848
categories:
  - Module Spotlight
  - PSRedfishEventListener
tags:
  - Modules

---
The [Redfish specification][1] supports event mechanism through which the target Redfish devices can send events from different components in the system to an event listener. The [PSRedfishEventListener][2] project provides an event listener that is create in native PowerShell.

Full documentation of commands with examples is available at: https://psredfishlistener.readthedocs.io/en/latest/

### Example Workflow

  1. Start an event listener using the `Start-PSRedfishEventListener` command.

```powershell
Start-PSRedfishEventListener -IPAddress 172.16.102.76 -Port 9090 -LogPath C:\RedfishEvents
```

The above command will start new event listener that is bound to a specific IP address on the local host and to port 9090. The log path will be set to C:\RedfishEvents.

  1. Perform registration on a Redfish Endpoint to send alerts to the listener. This is done using the `Register-PSRedfishEventSubscription` command.

```powershell
$credential = Get-Credential -Message 'Credentials to authenticate to the Redfish device ...'
Register-PSRedfishEventSubscription -EventDestination https://172.16.102.76 -IPAddress 172.16.100.21 -Credential $credential
```


The above command will register (create an event subscription) Redfish device 172.16.100.21 to send all event types to listener at https://172.16.102.76.

  1. Test event subscription using the`Send-PSRedfishTestEvent`.

```powershell
$credential = Get-Credential -Message 'Credentials to authenticate to the Redfish device ...'
Send-PSRedfishTestEvent -IPAddress 172.16.100.21 -Credential $credential -EventDestination https://172.16.102.76 -Verbose
```

The above command will submit a test event from Redfish device with an IP address 172.16.100.21 to the event listener at 172.16.102.76. The event type and message ID will be set to the defaults defined by the function.

4.Stop the event listener using the `Stop-PSRedfishEventListener` command.

```powershell
Stop-PSRedfishEventListener -IPAddress '172.16.102.76' -Verbose
```

I have a few new features lined up for the next release. Similar to PowerShell object events, with the upcoming release, you will be able to tag an action associated to a specific event type from all Redfish device source or a specific source.

[1]: https://www.dmtf.org/standards/redfish
[2]: https://github.com/rchaganti/PSRedfishEventListener