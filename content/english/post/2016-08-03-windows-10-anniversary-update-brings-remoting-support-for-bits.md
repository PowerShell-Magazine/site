---
title: Windows 10 Anniversary Update Brings Remoting Support for BITS
author: Jan Egil Ring
type: post
date: 2016-08-03T16:05:49+00:00
url: /2016/08/03/windows-10-anniversary-update-brings-remoting-support-for-bits/
post_views_count:
  - 3576
categories:
  - News
tags:
  - News

---
In this article we will look at a new feature in the BITS service which is included in Windows. Some background:

Background Intelligent Transfer Service (BITS) transfers files (downloads or uploads) between a client and server and provides progress information related to the transfers. You can also download files from a peer.

Use cases

  * Asynchronously transfer files in the foreground or background.
  * Preserve the responsiveness of other network applications.
  * Automatically resume file transfers after network disconnects and computer restarts.

Source: [Microsoft][1]

Possibly the most known service to leverage BITS is Windows Update, which even though it is downloading large amounts of data, rarely impacts the productivity of users since it is running asynchronous downloads in the background.

Introduced in Windows 7, the BitsTransfer PowerShell module contains several cmdlets for administering the BITS client:

![](/images/bits1.png)

This means we can leverage the benefits of BITS from PowerShell, as an alternative to using, for example, Invoke-WebRequest.

Previously, using cmdlets in the BITS module was not supported when invoked via PowerShell remoting. Let&#8217;s verify this on Windows 10 build 1511:

```powershell
Invoke-Command -ComputerName CLIENT-JR-02 -ScriptBlock {
  Write-Output "Testing remote BITS on PowerShell version $($PSVersionTable.PSVersion.ToString())"
    Start-BitsTransfer -Source 'https://github.com/janegilring/PSVersion/archive/master.zip' -Destination $env:TEMP -Asynchronous
}
```


![](/images/bits2.png)

We got an error message stating _“The remote use of BITS is not supported. For more information about BITS, see the MSDN documentation at_ [_http://go.microsoft.com/FWLINK/?LinkId=140888_][2]_”_.

There is also a Microsoft Connect [item][3] logged regarding this issue.

Now, let&#8217;s try the same thing against a Windows 10 computer running Windows 10 Anniversary Update:

![](/images/bits3.png)

This is good news, as BITS via PowerShell remoting is now supported in PowerShell 5.1. Most likely this will also be supported in Windows Server 2016, which will be released at the Microsoft Ignite conference at the end of September.

You can find additional details and examples in [this article][4] on MSDN.

As stated in the article, A BITS Job created from a Remote PowerShell session runs under that session’s user account context and will only make progress when there is at least one active local logon session or Remote PowerShell session associated with that user account. Therefore, we would need to add _-InDisconnectedSession_ to Invoke-Command in our example above for it to work properly. Alternatively we could have entered an interactive session using Enter-PSSession or setup a new PowerShell session using New-PSSession beforehand to run the BITS job in.

[1]: https://msdn.microsoft.com/en-us/library/windows/desktop/bb968799(v=vs.85).aspx
[2]: http://go.microsoft.com/FWLINK/?LinkId=140888
[3]: https://connect.microsoft.com/PowerShell/feedback/details/743030/make-it-easier-to-use-bits-with-powershell-remoting
[4]: https://msdn.microsoft.com/library/windows/desktop/ee663885.aspx#manage_ps_remote_sessions