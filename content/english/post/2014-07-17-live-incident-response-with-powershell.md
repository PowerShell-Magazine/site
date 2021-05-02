---
title: Live Incident Response with PowerShell
author: Emin Atac
type: post
date: 2014-07-17T16:00:41+00:00
url: /2014/07/17/live-incident-response-with-powershell/
categories:
  - infosec
  - security
tags:
  - infosec
  - security

---
We often see the offensive capabilities of PowerShell hit the headlines as it is more attractive. It’s good to know and see what attackers do to penetrate your network, execute code to remain undetected, move laterally, and steal administrative credentials. Scary? Paradoxically malicious code cannot hide as it needs to run. What can you do to protect yourself, your environment, critical assets and your business? The preparation phase of incident response is a key element in your defense.

First you should consider building barriers with multiple layers to prevent incidents to happen. How?

You should plan carefully the configuration of your environment to slow down attackers and limit the attack surface. You should also know your environment.

  * You should be able to build a whitelist of known good files. The built-in Get-FileHash cmdlet, introduced in PowerShell 4.0, can help. 

    ```
    Get-FileHash C:\Windows\system32\ntoskrnl.exe
    ```
    
    ![](/images/psautoruns1.png)
    
  * Have network segmentation, enable and configure firewalls not only at the edge of your perimeter but also on workstations. You can use Group Policies to deploy firewall rules or the built-in \*- NetFirewall\* cmdlets from the NetSecurity module. 

  ```
  Get-Command -Module NetSecurity
  ```

* Start to apply the least privilege principle by not turning off UAC (User Account Control) and let your users run their session with administrative privileges. If they run as standard users, the attacker will first have to trigger an elevation of privileges to gain administrative privileges.

  * Sign your code with a certificate so that you don’t have to lower the default execution policy of PowerShell. You can also use a Group Policy and Set-AuthenticodeSignature cmdlet. 

  ```
  Get-Help about_Execution_Policies
  ```

  * Whitelist what can be run. If you have Applocker, you should definitely leverage that control. PowerShell has a built-in Applocker module since version 2.0. 

  ```
  Get-Command -Module Applocker
  ```

  ![](/images/psautoruns2.png)

  If you’ve signed your code, you can use Publisher-based rules in Applocker:

  ![](/images/psautoruns3.png)

  * Deploy an audit policy to be able to collect and monitor events.
  * Consider using the JEA (Just Enough Admin) kit to ease the deployment of WinRM constrained endpoints

  * Etc&#8230;

    While focused on PowerShell, there are other defenses that you should implement (like a deploying an Antivirus, EMET (Enhanced Mitigation Toolkit)&#8230; Please also note that the above list is far from being exhaustive and that these controls can be bypassed, defeated… </ul> 

    Having good defense in your “fortress“ isn’t enough as it can be breached from the inside.

    Zero-day or unknown vulnerabilities still exist but it&#8217;s more likely that your users will be targeted as they are the weakest point in your environment even if you have the best security trainings and awareness campaigns. So, **you should assume breach** and that your Antivirus product won’t recognize the threat. How do you defend yourself in this case?

    The second key element in the preparation phase for incident response is that you should consider building the capability to quickly respond to incidents.

    You should have a process where you can quickly run code to hunt for malicious activity in your environment. To spot the malicious activity, you should first know what normal activity look like.

    To quickly run code, you can leverage both PowerShell remoting and workflows introduced in PowerShell 3.0 to parallelize the workload. An alternative to workflows are runspaces.

    Can I do live incident response with PowerShell? Of course. The SANS published a white paper on <a href="http://www.sans.org/reading-room/whitepapers/forensics/live-response-powershell-34302" target="_blank">Live Response using PowerShell</a>. PowerShell MVP <a href="https://twitter.com/mattifestation" target="_blank">Matt Graeber</a> also developed some tools to help us do <a href="http://www.exploit-monday.com/2012/03/powershell-live-memory-analysis-tools.html" target="_blank">live memory analysis</a>.

    You can also track indicators of compromise that uses persistence in the registry. A traditional well-known tool for this purpose is autoruns.exe from Sysinternals.

    To demonstrate what can be done to investigate malware persistence, I wrote a PowerShell function that will go through the 192 launch points. You can find the code <a href="https://gist.github.com/p0w3rsh3ll/b093e3dca1e41cf451a2" target="_blank">here</a>.

    Get-PSAutoRun uses the same categories as autoruns.exe: Logon, Explorer Add-ons, Scheduled Tasks, Services, Drivers, Codecs, LSA Providers,&#8230; just to name a few.

    ![](/images/psautoruns4.png)

    You can choose to either get everything using the -All parameter:

    <pre class="brush: powershell; title: ; notranslate" title="">Get-PSAutorun -All | Format-Table -Property Path,ImagePath,Category 
    </pre>

    ![](/images/psautoruns5.png)

    or specify a list of switches that match one or more categories:

    <pre class="brush: powershell; title: ; notranslate" title="">Get-PSAutorun -Logon -LSAsecurityProviders | Format-Table -Property Path,ImagePath,Category
    </pre>

    Whenever you specify the -All parameter, any specific category you mentioned is ignored allowing to return all autoruns.

    There are also two additional parameters that you may leverage when investigating malware persistence. You may want to get MD5, SHA1 and SHA2 hashes of files and know whether they are digitally signed or not.

    <pre class="brush: powershell; title: ; notranslate" title="">Get-PSAutorun -All -ShowFileHash -VerifyDigitalSignature
    </pre>

    ![](/images/psautoruns6.png)

    You get the MD5, SHA1 and SHA256 properties when you specify the -ShowFileHash parameter and the Signed property with the -VerifyDigitalSignature parameter.

    What’s next? Malware persistence isn’t the only thing to look at. You should check for files like the hosts file in C:\Windows\system32\drivers\etc\ being hijacked. You should look for recent suspicious events in the logs like the “audit log was cleared”. You can also check if the firewall is on and if the rules enabled match what is in your baseline of known good rules.

    All this post-mortem controls aren’t enough to keep the bad guys out of your network if you don’t have a good patch management policy in place to fix known vulnerabilities in the operating system and all your applications. You can find a list of 20 Critical Security Controls for Effective Cyber Defense on this page: <a href="http://www.sans.org/critical-security-controls" target="_blank">http://www.sans.org/critical-security-controls</a>.

    My wish for the future: I’d like to see more people sharing code about how they defend their network or write PowerShell code for forensics and live incident response.