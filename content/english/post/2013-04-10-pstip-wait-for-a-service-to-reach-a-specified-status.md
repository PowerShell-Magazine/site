---
title: '#PSTip Wait for a Service to reach a specified status'
author: Shay Levy
type: post
date: 2013-04-10T18:00:00+00:00
url: /2013/04/10/pstip-wait-for-a-service-to-reach-a-specified-status/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
So, in your script, you need to wait for a service until it reaches a specified status and performs an action based on the new state. One way to achieve this, and an ineffective one, would be to poll the status using a while loop:

```
$svc = Get-Service W3SVC
while($svc.State -ne 'Stopped')
{
	Start-Sleep -Seconds 1
}

... do your thing here...
```

Instead, you could wait for the service to reach the specified status using [one of its native methods][1]:

<pre class="brush: powershell; title: ; notranslate" title="">$svc.WaitForStatus('Stopped')
</pre>

This will wait infinitely for the service to reach the specified state, and script execution is halted until the service state changes. Waiting forever for the service to change its state may not be what we want to do, so instead we can use the [second overload of the _WaitForStatus_ method][2] and specify an expiration time-out value.

```
# wait for 5 seconds
$svc.WaitForStatus('Stopped','00:00:05')

... the rest of the script ...
```

[1]: http://msdn.microsoft.com/en-us/library/system.serviceprocess.servicecontroller.waitforstatus.aspx
[2]: http://msdn.microsoft.com/en-us/library/35st9aw1.aspx