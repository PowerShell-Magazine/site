---
title: Using Copy-VMFile cmdlet in Windows Server 2012 R2 Hyper-V
author: Ravikanth C
type: post
date: 2013-12-16T17:00:44+00:00
url: /2013/12/16/using-copy-vmfile-cmdlet-in-windows-server-2012-r2-hyper-v/
categories:
  - How To
  - Hyper-V
tags:
  - Hyper-V
  - How To

---
With Windows Server 2012 R2, the Hyper-V product team introduced a new cmdlet called _Copy-VMFile_. This, as the name implies, help you copy a file into a Hyper-V Virtual Machine (VM). A much needed functionality, in my opinion. The syntax for copying a file is simple.

<pre class="brush: powershell; title: ; notranslate" title="">Copy-VMFile "WC7-1" -SourcePath C:\Scripts\test.txt -DestinationPath C:\Scripts\test.txt -FileSource Host
</pre>

For the above operation to succeed, VM must have the C:\Scripts folder. If not, you will see an error that the system cannot find the file specified. This is where the _-CreateFullPath _parameter can be used to create the destination folder if it does not exist.

<pre class="brush: powershell; title: ; notranslate" title="">Copy-VMFile "WC7-1" -SourcePath C:\Scripts\test.txt -DestinationPath C:\Scripts\test.txt -FileSource Host -CreateFullPath
</pre>

Now, there are certain things you need to know before you start using this.

### #1. Update Integration Service Components

Like I mentioned earlier, this cmdlet is a part of Windows Server 2012 R2 Hyper-V release. So, the guest OS must be running the same level of integration components. You can use the Hyper-V cmdlet _Get-VM_ to check the version of the integration services. Here is a sample snippet I use.

<pre class="brush: powershell; title: ; notranslate" title="">Get-VM | Select Name, IntegrationServicesVersion, @{Name="IsUpdateNeeded";Expression={$_.IntegrationServicesVersion -lt [version]'6.3.9600.16384'}}
</pre>

![](/images/ictest.png)

### #2. Ensure Guest Service Interface component is enabled

Once you verify that the integration services version is 6.3.9600.16834, proceed to check if the Guest Service Interface component in the integration services is enabled or not. Again, we can do this using the Hyper-V cmdlets. This service must be enabled if you want to use the _Copy-VMFile_ cmdlet to copy files into a VM.

<pre class="brush: powershell; title: ; notranslate" title="">Get-VM | Get-VMIntegrationService -Name "Guest Service Interface" | Select VMName, Enabled
</pre>

![](/images/gis.png)

As you see in the above output, the Guest Service Interface component is not enabled on WS08-R2-1 virtual machine. Let us see what happens when I try to copy a file.

![](/images/copyerror.png)

You can enable this integration service component by using the _Enable-VMIntegrationService_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">Get-VM WS08-R2-1 | Get-VMIntegrationService -Name "Guest Service Interface" | Enable-VMIntegrationService -Passthru
</pre>

After enabling the Guest Service Interface component in the VM, you should be able to copy a file from host to guest.

### #3. What if you are still unable to copy the file into a VM?

Yes, I know, there is a chance. If you still encounter issues while using the _Copy-VMFile_ cmdlet, check the integration services status inside the guest. For this, run the following command in the guest Operating System.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Service -Name *vm*
</pre>
![](/images/services.png)

As you see, if the <em>vmicguestinterface</em> service is stopped inside the guest OS, we won&#8217;t be able to use <em>Copy-VMFile</em> cmdlet to copy files. So, all you need to do is start the service and ensure it is set to automatic start mode.

