---
title: '#PSTip Using Start-BitsTransfer and waiting for download to complete'
author: Ravikanth C
type: post
date: 2014-01-17T19:00:09+00:00
url: /2014/01/17/pstip-using-start-bitstransfer-and-waiting-for-download-to-complete/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

There was a quick status this morning on Facebook where my friend Deepak Dhami a.k.a [Dexter][1] posted a quick snippet that read something like this:

![](/images/deepak.png)

I looked at this and thought, there is a better PowerShell way! 🙂

We both started working out our solutions and this is what I came up with:

```
$uri = "http://download.microsoft.com/download/D/0/F/D0F564A3-6734-470B-9772-AC38B3B6D8C2/dotNetFx45_Full_x86_x64.exe"
$job = Start-BitsTransfer -Source $uri -Destination C:\Downloads -Asynchronous

While ($job.JobState -eq "Transferring") {
    Sleep -Seconds 1
}

If ($job.InternalErrorCode -ne 0) {
    ("Error downloading the file {0}" -f $job.InternalErrorCode) | Out-File C:\downloads\downloaderror.log
} else {
    #Do something here
    #Stop-Computer -Force
}
```

This is what Deepak worked out:

```
$job = Start-Job -Name DownloadJob -ScriptBlock { Start-BitsTransfer -Source "http://cdimage.kali.org/kali-latest/amd64/kali-linux-1.0.6-amd64.iso" -Destination "C:\temp\KaliLinux.iso" -DisplayName "Download" -Asynchronous }
$action = {
    if ($job.State -eq "completed")
    {
        if ((Get-BitsTransfer -Name Download).JobState -eq "Transferred")
        {
            Write-host -ForegroundColor Gray "Download Complete ...Halting Machine"
            Stop-Computer -Force
        }
    }
}
Register-ObjectEvent -InputObject $job -EventName StateChanged -Action $action
```


Now, both are perfectly valid solutions. If I were to use the eventing way of doing this, I&#8217;d use the _Start-Job_ cmdlet to create a background job. The BITS job generated by using _-Asynchronous_ switch has no events that we can subscribe to. So, I had to fall back on using a While loop.

How do you do this?

[1]: https://www.facebook.com/DexterPOSH