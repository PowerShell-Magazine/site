---
title: '#PSTip Automatic refresh of Hyper-V VM Objects'
author: Ravikanth C
type: post
date: 2013-03-05T19:00:44+00:00
url: /2013/03/05/pstip-automatic-refresh-of-hyper-v-vm-objects/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

When using the Hyper-V PowerShell cmdlets in Windows Server 2012, the VM objects can be retrieved using _Get-VM_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">$vm = Get-VM
</pre>

By default, these VM objects refresh based on the events from the virtual machines. For example, take a look at this screen capture:

![](/images/vm.png)

As you see, the VM object in _$vm_ gets refreshed automatically. This is possible because, by default, virtual machine eventing is enabled. We can disable this behavior by using _Disable-VMEventing_ cmdlet. This might be useful in cases when we want to use the VM object in a script, work on the VM&#8217;s captured object state and not refresh the object.