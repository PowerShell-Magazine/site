---
title: '#PSTip A hidden gem of the Test-Connection cmdlet'
author: Shay Levy
type: post
date: 2012-08-26T10:02:53+00:00
url: /2012/08/26/pstip-a-hidden-gem-of-the-test-connection-cmdlet/
views:
  - 8376
post_views_count:
  - 1639
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
PowerShell equivalent of the ping.exe utility is the Test-Connection cmdlet. One of the hidden and less known capabilities of Test-Connection is the abitlity to ping a computer using another computer as the source. For example, when troubleshooting, you often need to verify connectivity between two computers using a quick ping test. The common practice is to make a remote desktop connection to ComputerA, open cmd, then ping ComputerB.

With the Test-Connection cmdlet you are no longer required to do so. You can use the cmdlet to initiate a ping check that uses ComputerA as the source computer and ComputerB as the destination.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Test-Connection -Source ComputerA -ComputerName ComputerB
</pre>

**Note**: WMI is the underlying mechanism that Test-Connection is built upon and this method requires the caller to have administrator&#8217;s rights on the source computer and the source computer&#8217;s OS version must be Windows XP and above.