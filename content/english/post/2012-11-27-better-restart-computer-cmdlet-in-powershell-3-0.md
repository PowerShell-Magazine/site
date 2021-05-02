---
title: Better Restart-Computer cmdlet in PowerShell 3.0
author: Vinith Menon
type: post
date: 2012-11-27T17:00:11+00:00
url: /2012/11/27/better-restart-computer-cmdlet-in-powershell-3-0/
categories:
  - How To
  - Tips and Tricks
tags:
  - How To
  - Tips and Tricks

---
As the name suggests the Restart-Computer cmdlet helps in restarting the operating system on the local and remote computers.

Compared to earlier version of Restart-Computer in PowerShell 2.0, the new Restart-Computer cmdlet offers much better flexibility and control to an admin. PowerShell Scripts which require an intermittent restart of remote computers in between of a script execution are handled with better control in the new version of this cmdlet. I went through all Restart-Computer’s parameters in PowerShell 2.0 and 3.0 and here’s what I saw:

&#8211; in PowerShell 3.0, Restart-Computer has 15 parameters excluding the common parameters

![](/images/Vinith1.png)

&#8211; PowerShell 2.0 has a total of 9 parameters for Restart-Computer cmdlet

![](/images/Vinith2.png)

I&#8217;ve prepared a small Excel sheet to compare the various new and old parameters present in Restart-Computer.

![](/images/Vinith15.jpg)

PowerShell 3.0 has 6 more new parameters for Restart-Computer and the Authentication parameter is renamed to DcomAuthentication.

Restart-Computer cmdlet allows us to run the restart operation as a background job. We can also specify the authentication levels and provide alternate credentials to initiate the restarts.

One of the brilliant features of this cmdlet in Windows PowerShell 3.0 is that we can wait for the restart to complete before running the next command, specify a waiting timeout and query interval, and wait for particular services to be available on the restarted computer. This feature makes it practical to use Restart-Computer in scripts which require a computer restart in between of its execution.

We can also use WSMan protocol to restart the computer, in case DCOM calls are blocked by a Firewall rule or an enterprise policy. This feature was not available in PowerShell 2.0. Now, let’s talk about some of the cool features available with the new parameter sets introduced in PowerShell 3.0.

### -Wait

We can use this parameter in a script to restart computers and then continue processing when the restart is complete.

By default, the Wait parameter waits indefinitely for the computers to restart, but you can use the Timeout parameter to specify the duration of wait and the For and Delay parameters to wait for particular services to be available on the restarted computers. The following PowerShell one-liner represents an example for this parameter:

<pre class="brush: powershell; title: ; notranslate" title="">Restart-Computer -ComputerName Server01 -Wait
</pre>

This command restarts the Server01 remote computer and waits indefinitely for the remote server to restart. By default it checks for WMI, WinRM, and PowerShell connectivity to move to the next line in script.

Here’s an example when I restarted one of my server, by default it checked till WMI, WinRM, and PowerShell connectivity was established to return me with the PowerShell prompt.

![](/images/Vinith6.png)

![](/images/Vinith7.png)

### -For

When we specify the For parameter with Restart-Computer it waits until the specified service or feature is available after the computer is restarted for a set of predefined values. This parameter is valid only with the Wait parameter. Valid values are:

  * Default: Waits for Windows PowerShell to restart.
  * PowerShell: Can run commands in a Windows PowerShell remote session on the computer.
  * WMI: Receives a reply to a Win32_ComputerSystem query for the computer.
  * WinRM: Can establish a remote session to the computer by using WS-Management.

Now the new ISE in PowerShell 3.0 has IntelliSense which auto-populates these values:

![](/images/Vinith8.png)

The next PowerShell one-liner represents an example for this parameter:

<pre class="brush: powershell; title: ; notranslate" title="">Restart-Computer -ComputerName Server01 -Wait -For WinRM
</pre>

This command restarts the Server01 remote computer and waits until WinRM service is up and running on the remote server.

![](/images/Vinith9.png)

### -Timeout

Specifies the duration of the wait, in seconds. When the timeout elapses, Restart-Computer returns the command prompt, even if the computer is not restarted. The default value, -1, represents an indefinite timeout. The Timeout parameter is valid only with the Wait parameter.

I specified a timeout of 10 seconds for a computer restart, as my computer did not restart in 10 seconds and took much longer time I was immediately returned to the PowerShell prompt:

<pre class="brush: powershell; title: ; notranslate" title="">Restart-Computer -ComputerName Server01 -Wait -For WinRM -Timeout 10
</pre>

![](/images/Vinith10.png)

### -Delay

This parameter determines how often Windows PowerShell queries the service that is specified by the For parameter to determine whether it is available after the computer is restarted.  The default value is 5 (seconds).This parameter is valid only with the Wait and For parameters.

With the below PowerShell one-liner I have illustrated the same with two screenshots representing the progress of restarting process. I have specified a delay of 6 seconds; so after a delay of every  6 seconds PowerShell queries for WinRM connectivity to server until it’s able to verify that  connectivity has been successfully established.

<pre class="brush: powershell; title: ; notranslate" title="">Restart-Computer -ComputerName Server01 -Wait -For WinRM -Delay 6
</pre>

![](/images/Vinith11.png)

![](/images/Vinith12.png)

If we specify a lower Delay parameter it reduces the interval between queries to the remote computer that determine whether it is restarted.

### -Protocol

Specifies which protocol to use to restart the computers. Valid values are WSMan and DCOM. The default value is DCOM.  These settings are designed for enterprises in which DCOM-based restarts fail because DCOM is blocked, such as by a firewall rule.

![](/images/Vinith13.png)

<pre class="brush: powershell; title: ; notranslate" title="">Restart-Computer -ComputerName Server01 -Protocol WSMan
</pre>

This command restarts the Server01 remote computer and uses the WSMan protocol.

### -WsmanAuthentication

Specifies the mechanism that is used to authenticate the user&#8217;s credentials when using the WSMan protocol. Valid values are Basic, CredSSP, Default, Digest, Kerberos, and Negotiate. The default value is _Default_.

![](/images/Vinith14.png)

<pre class="brush: powershell; title: ; notranslate" title="">Restart-Computer -ComputerName Server01 -WSManAuthentication Kerberos
</pre>

This command restarts the Server01 remote computer and uses Kerberos authentication. If the User does not have the permissions to restart the remote server it would throw an access denied error.