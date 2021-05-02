---
title: 'Accidental Sabotage: Beware of CredSSP'
author: Joe "clymb3r" Bialek
type: post
date: 2014-03-06T17:00:30+00:00
url: /2014/03/06/accidental-sabotage-beware-of-credssp/
categories:
  - How To
  - Security
tags:
  - How To
  - Security

---
One common issue that an administrator faces when using PowerShell remoting is the “double hop” problem. An administrator uses PowerShell remoting to connect to Server A and then attempts to connect from Server A to Server B. Unfortunately, the second connection fails.

The reason is that, by default, PowerShell remoting authenticates using a “Network Logon”.  Network Logons work by proving to the remote server that you have possession of the users credential without sending the credential to that server (see [Kerberos][1] and [NTLM][2] authentication). Because the remote server doesn&#8217;t have possession of your credential, when you try to make the second hop (from Server A to Server B) it fails because Server A doesn’t have a credential to authenticate to Server B with.

To get around this issue, PowerShell provides the CredSSP ([Credential Security Support Provider][3]) option. When using CredSSP, PowerShell will perform a “Network Clear-text Logon” instead of a “Network Logon”. Network Clear-text Logon works by sending the user’s clear-text password to the remote server. When using CredSSP, Server A will be sent the user’s clear-text password, and will therefore be able to authenticate to Server B. Double hop works!

![](/images/credssp1.png)

Figure 1: The computers colored red have the user credentials cached on them.

While this is certainly convenient, it comes at a price: If the server you authenticate to using CredSSP is compromised, so are your credentials.

An attacker with **administrative privilege** on a server can intercept any data that is sent to/from the server, as well as view any data in memory and on disk, by design. Because the CredSSP authentication option sends your clear-text credentials to the remote server, an attacker with administrative privilege on the remote server can easily intercept your username and password.

To prove this to you, I’ll do a quick demonstration.

First, I’ll create a PSSession to connect to a remote session using CredSSP; then I will run [Mimikatz][4], a hacker tool used to capture the credentials of users logged in to the server. Because this is PowerShell Magazine, I will use a version of Mimikatz, called [Invoke-Mimikatz][5], which has been modified to run as a PowerShell script.

![](/images/credssp2.jpg)

Figure 2: Connecting to a server using CredSSP and running Invoke-Mimikatz

![](/images/credssp3.png)

Figure 3: Mimikatz output showing that the credentials for DEMO\administrator are stored on a remote server when using CredSSP

Next, I’ll create a PSSession to connect to a remote computer without using CredSSP. This time, Mimikatz isn’t able to capture any credentials.

![](/images/credssp4.png)

Figure 4: Connecting to a server without using CredSSP and running Invoke-Mimikatz

This shows that when you use CredSSP, your credentials can be captured on the remote computer. If you don’t use CredSSP, you can authenticate to any server you’d like without disclosing your credentials (which makes it harder for an attacker to obtain privileged credentials).

Is it ever safe to use CredSSP? Certainly. The important thing to realize is that you are putting your credentials on the server you authenticate to. It is a bad idea to use CredSSP to authenticate to a user’s workstation using a domain administrator account; you are essentially giving away the keys to the kingdom. It would be perfectly acceptable to use a domain administrator to authenticate to a domain controller using CredSSP because the domain controller is a high trust server.

Designing safe operating procedures is outside the scope of this article, but the general rule is: Don’t put high trust credentials on low trust computers. Additionally, you should always try to design your systems to work with single-hop rather than double-hop so that CredSSP isn’t needed.

**Update:** This testing was done using Windows Server 2012. Microsoft has made changes to Windows Server 2012R2 and Windows 8.1 to eliminate clear-text credentials from being stored in memory. This means that an attacker who runs Mimikatz will no longer see your clear-text credentials. An attacker will still see your NT password hash and your Kerberos TGT, both of which are password equivalent and can be used to authenticate as you over the network.

Additionally, even though your clear-text credential is not saved in memory, it is still sent to the remote server. An attacker can inject malicious code in the Local Security Authority Subsystem Service (LSASS.exe) and intercept your password in transit. So while you may not see your password with Mimikatz anymore, your password can still be recovered by an attacker.

[1]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa378747(v=vs.85).aspx
[2]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa378749(v=vs.85).aspx
[3]: http://support.microsoft.com/kb/951608
[4]: http://blog.gentilkiwi.com/mimikatz
[5]: https://github.com/clymb3r/PowerShell/blob/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1