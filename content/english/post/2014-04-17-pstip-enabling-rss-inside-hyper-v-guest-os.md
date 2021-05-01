---
title: '#PStip Enabling RSS inside Hyper-V guest OS'
author: Emin Atac
type: post
date: 2014-04-17T18:00:01+00:00
url: /2014/04/17/pstip-enabling-rss-inside-hyper-v-guest-os/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

While reviewing results returned by the Best Practices Analyzer on a virtual machine, a warning about RSS (Receive Side Scaling) was raised.

![](/images/image00111.jpg)

RSS has been around since Windows 2008. The online help states:

_“RSS is a scalability technology that distributes the receive network traffic among multiple processors by hashing the header of the incoming packet. Without RSS in Windows Server® 2008 and later, network traffic is received on the first processor which can quickly reach full utilization limiting receive network throughput.”_

It’s a nice surprise&#8211;RSS has now been extended to Windows Server 2012 R2 Hyper-V. The guest virtual machine can now use this functionality. To check whether my virtual network adapter was really RSS capable, I used the _Get-SmbClientNetworkInterface_ cmdlet:

<pre class="brush: powershell; title: ; notranslate" title="">Get-SmbClientNetworkInterface |
Format-Table -Property FriendlyName,RssCapable -AutoSize
</pre>
![](/images/image00211.png)

To determine how the virtual network adapter was currently configured:

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetAdapter | Where Status -eq "Up" |
Get-NetAdapterAdvancedProperty -DisplayName "Receive Side Scaling" |
Format-Table -AutoSize
</pre>

![](/images/image00311.png)

To turn on RSS on the RSS-capable virtual network adapter:

<pre class="brush: powershell; title: ; notranslate" title="">Get-NetAdapter | Where Status -eq "Up" |
Get-NetAdapterAdvancedProperty -DisplayName "Receive Side Scaling" |
Where DisplayValue -eq "Disabled" |
Enable-NetAdapterRss -Verbose
</pre>
![](/images/image0041.png)

Note that you can add _–NoRestart_ switch to the _Enable-NetAdapterRss_  cmdlet to avoid the network adapter from being restarted immediately.