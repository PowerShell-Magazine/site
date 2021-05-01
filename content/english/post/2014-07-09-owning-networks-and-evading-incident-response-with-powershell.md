---
title: Owning Networks and Evading Incident Response with PowerShell
author: Joe "clymb3r" Bialek
type: post
date: 2014-07-09T16:00:03+00:00
url: /2014/07/09/owning-networks-and-evading-incident-response-with-powershell/
categories:
  - InfoSec
  - Security
tags:
  - InfoSec
  - Security
---
Up until several months ago, I was a member of a penetration test team tasked with compromising data centers and evading detection. Industry standard tools such as <a href="http://www.metasploit.com" target="_blank">Metasploit </a>(an attack toolkit that includes a backdoor named Meterpreter) and <a href="http://blog.gentilkiwi.com/mimikatz" target="_blank">Mimikatz </a>(a password dumper) worked well, but I was a paranoid attacker and was worried that running these tools (compiled as unsigned EXE files) would get me detected.

I started to take a look at PowerShell after reading a blog post by Matt Graeber on <a href="http://www.exploit-monday.com/2012/08/powersploit-inject-shellcode-update.html" target="_blank">launching Meterpreter using PowerShell</a>. Since antivirus pays no attention to PowerShell scripts, I was able to use Meterpreter without launching a suspicious EXE and without having to worry about disabling antivirus.

I wanted to go a little further though. Instead of just loading Meterpreter, I wanted to be able to load unmanaged DLLs and EXEs (both are actually Windows PE files) without calling LoadLibrary or CreateProcess (because these APIs can be monitored by AppLocker and other similar tools).

The solution to this problem was to write my own PE loader. Instead of relying on Windows APIs (LoadLibrary, CreateProcess) to load PE files in memory, I wrote a PowerShell script Invoke-ReflectivePEInjection that roughly recreates the functionality provided by the Windows API. The benefits of Invoke-ReflectivePEInjection are over the Windows APIs are:

  1. The PE file doesn’t need to be written on disk, it can be supplied as a byte array
  2. No logging or application whitelisting will be done on process creation or DLL loads because Windows sees nothing
  3. Antivirus will not currently detect this

I gave a short talk at DEF CON on the details of this script that can be found <a href="http://www.youtube.com/watch?v=OAd68_SYQc8" target="_blank">here</a>:

{{< youtube OAd68_SYQc8 >}}

I also have several blog posts that dive in the details of <a href="http://clymb3r.wordpress.com/2013/04/06/reflective-dll-injection-with-powershell" target="_blank">the script</a>. I treat _Invoke-ReflectivePEInjection_ like a Swiss Army knife, and have used it to create a few more scripts:

  1. _**Invoke-Mimikatz**_: This combines Invoke-ReflectivePEInjection with Mimikatz to provide an easy to use PowerShell script that can dump the credentials for all users logged in to a system.
  2. _**Invoke-NinjaCopy**_: This script packages an NTFS file system parser in to a PowerShell script. Sometimes you need access to a file but another process has an exclusive access to the file, preventing you from accessing it. If you are an administrator, you can open a handle to the volume the file is on (ex: C:\ volume) and parse the NTFS file system data structures manually to identify the location of the raw data you need on the volume. You can then directly copy the data off the volume without ever opening a handle to the file in question. Invoke-NinjaCopy allows you to copy files off of remote systems over PowerShell remoting (meaning you write no files to disk on the remote system). This is useful for attackers and defenders (collecting forensic evidence). More information on this script can be <a href="http://clymb3r.wordpress.com/2013/06/13/using-powershell-to-copy-ntds-dit-registry-hives-bypass-sacls-dacls-file-locks" target="_blank">found here</a>.
  3. **_Invoke-CredentialInjection_:** This is a script that can be used to call the Windows API “LsaLogonUser” from within a process of the attackers choosing. Windows logs which processes are calling the logon APIs and sometimes investigators can detect attacker activity by looking for logons happening from non-standard processes. This script allows an attacker to create a logon originate from any process on the system. For more information about the script and what specifically it is used for, you can read my blog post <a href="http://clymb3r.wordpress.com/2013/11/17/injecting-logon-credentials-with-powershell" target="_blank">here</a>.

I have also created scripts unrelated to PE injection.

One such script, Invoke-TokenManipulation, allows an administrator to enumerate the logon tokens for all users logged on to the system. It then allows the administrator to create new processes using any of the tokens impersonated (which effectively makes the process run under the other users account, but on the desktop of the administrator who created the process). This script is similar to the tool <a href="https://labs.mwrinfosecurity.com/blog/2012/07/18/incognito-v2-0-released/" target="_blank">Incognito</a>. For more information about _Invoke-TokenManipulation_, see <a href="http://clymb3r.wordpress.com/2013/11/03/powershell-and-token-impersonation" target="_blank">my blog</a>.

Although I no longer do penetration testing against data centers, I do still maintain these tools. Financially motivated or state sponsored attackers can have large budgets for their toolkits, but penetration testers usually do not. For this reason, it is important for penetration testers to have free, quality tools available to test with to simulate the threat of advanced and well-funded adversaries.

I now work for the Microsoft Security Response Center on the REACT team. I’m responsible for assessing vulnerabilities reported to Microsoft (and 0-days exploited in the wild), finding variants, and doing software penetration tests on high value components. In my new role, I use PowerShell to automate tasks such as configuring servers used for reproducing bugs and automating things such as fuzzing runs.

As a final note, all of the tools mentioned above can be found both on my <a href="https://github.com/clymb3r" target="_blank">personal GitHub account</a>, and as part of the <a href="https://github.com/mattifestation/PowerSploit" target="_blank">PowerSploit toolkit</a>.