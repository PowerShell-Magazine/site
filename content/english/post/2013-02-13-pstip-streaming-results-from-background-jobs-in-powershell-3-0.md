---
title: '#PSTip Streaming results from background jobs in PowerShell 3.0'
author: Ravikanth C
type: post
date: 2013-02-13T19:01:35+00:00
url: /2013/02/13/pstip-streaming-results-from-background-jobs-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

PowerShell background jobs are very powerful way to achieve parallelism when performing administrative tasks. When we use background jobs, the results from the tasks in the background job remain in the background runspace. We need to use the _Receive-Job_ cmdlet to be able to see the results or output from the background jobs. But, what if we want to see the results as they get generated?

Let us consider a hypothetical example:

<pre class="brush: powershell; title: ; notranslate" title="">$jobs = "vm1","vm2","vm3" | foreach {
    Start-Job -ScriptBlock { New-VM -Name $using:_ -Verbose }
}
</pre>

The above example creates three Hyper-V virtual machines using background jobs so that we speed up the VM creation process. Now, how can we stream the results as the VMs are created?

Here is a simple method:

<pre class="brush: powershell; title: ; notranslate" title="">while ($jobs.HasMoreData -and $jobs.State -eq "Running") {
    Receive-Job -Job $jobs
}
</pre>
If you run the above two snippets together, you should see something similar to what is shown below:

![](/images/jobs1.png)

The order of these messages is certainly not guaranteed because of the nature of background jobs and parallelism. By default, the results received will be deleted from the background runspace unless we specify the `-Keep` parameter. If you prefer to use the results apart from just streaming them, you may want to consider using the `-Keep` parameter.