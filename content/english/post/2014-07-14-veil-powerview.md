---
title: Veil-PowerView
author: Will Schroeder
type: post
date: 2014-07-14T16:00:33+00:00
url: /2014/07/14/veil-powerview/
categories:
  - infosec
  - security
  - Modules
  - Veil-PowerView
tags:
  - infosec
  - security
  - Modules
  - Veil-PowerView
---
I was led to PowerShell in the past few years as it began to rise to prominence in the information security community. As a penetration tester and red teamer for Veris Group&#8217;s Adaptive Threat division, my job depends on keeping abreast of current attack tools, tactics, techniques and procedures, and PowerShell is definitely the &#8216;new hotness&#8217; in infosec. It&#8217;s been gaining traction ever since the seminal DefCon 18 talk [_PowerShell OMFG_][1] given by Dave Kennedy and Josh Kelly in 2010, and an increasing number of attack toolsets, public and private, have been ported to the language. To clarify, PowerShell does not help attackers actively exploit or compromise systems, but it does assist with the phase of the attack cycle we in the security community call [post-exploitation][2]. Once an attacker gains control of a machine, the availability of the PowerShell language provides an excellent environment to perform a variety of actions that can help turn that compromise of that single system into the compromise of an entire domain.

[Matt Graeber][3]&#8216;s post _&#8220;_[_Why I Choose PowerShell as an Attack Platform_][4]_&#8220;_ articulates better than I could why PowerShell is great for post-exploitation. Essentially, Microsoft has given attackers a means to run fully-featured attacker toolsets (or [malware][5]) on any modern Windows operating system by providing a whitelisted, AV-safe environment with access to the full .NET Framework. Being able to dynamically load PowerShell scripts into memory without touching disk is an added plus, allowing us as attackers to leave as forensically small a footprint as possible on an exploited machine.

My contribution to the offensive PowerShell community, [Veil-PowerView][6], is a component of the open source attack toolkit I co-founded called the Veil-Framework. The toolset started with Veil-Evasion, which generates AV-evading executables, branched into payload delivery with Veil-Catapult, and then moved into situational awareness with Veil-PowerView. As we began to move into more advanced engagements, we began to focus more on PowerShell as a post-exploitation tool; Veil-PowerView is the result of gap areas we encountered between our operational goals and the capabilities of current public tools.

PowerView&#8217;s inspiration initially came from an experience on an assessment where a client had implemented an interesting defense- the disabling of all &#8220;net *&#8221; commands on domain machines. Initially this threw a wrench in some of our normal post-exploitation activities, however during our post assessment breakdown we started brainstorming ways around this particular defense in case we encountered it again. Bypassing it completely ended up being trivial through the use of PowerShell’s DirectoryServices.ActiveDirectory namespace and Active Directory Service Interfaces:

![](/images/vimage0011.png)

PowerView&#8217;s _Get-Net*_ cmdlets now handle everything from retrieving the local domain/user, querying Active Directory for domain user and group information, to adding local or domain users and more:

![](/images/vimage0011.png)

This was the initial motivation for the tool, a contingency in case we encountered a situation again where _net.exe_ was disabled. The next phase of development was inspired by [Rob Fuller&#8217;s][7] _netview.exe_ project, which &#8220;[utilizes various Windows API calls to perform enumeration across network machines][8]&#8220;. PowerView&#8217;s port of _netview.exe_, _Invoke-Netview_, will query Active Directory for machines using ADSI, and then for each machine will enumerate sessions using a Win32-api implementation of [NetSessionEnum][9], logged on users using a Win32-api implementation of [NetWkstaUserEnum][10], and available shares using a Win32-api implementation of [NetShareEnum][11]. The Windows API calls were later rewritten using information from Matt Graeber&#8217;s post [_&#8220;Accessing the Windows API in PowerShell via internal .NET methods and reflection&#8221;_][12]. This allows PowerView to operate completely in memory, as it doesn’t rely on _csc.exe_ to temporarily compile C# code in order to access the Windows API.

![](/images/vimage0011.png)

From there, a few related functions were built to hunt for users on a domain. _Invoke-UserHunter_ will query Active Directory for users of a particular group, say _“Domain Admins”_, and then utilizes similar techniques as _Invoke-Netview_ to enumerate loggedon users and users with active sessions, matching up where target users are located. _Invoke-StealthUserHunter_ takes a more subtle, &#8216;red-teamy&#8217; approach to hunting down user locations. It will then extract all servers found from user _HomeDirectory_ entries, and run a _Get-NetSessions_ call against each found file server to get current user sessions on those targets, again matching up where target users are located. Both of these functions are among the most useful for us on security engagements: once we find a way to access target machines, say through local admin password reuse, one of the things that used to be tedious was finding where high value users were logged in. With _PowerView_, we can now automate and accelerate this previously painful process:

![](/images/vimage0011.png)

From there, more and more functionality has been added into PowerView as gaps have arisen for us on assessments between what we want to accomplish and the functionality of current toolsets. _PowerView_ can now query AD for machines likely vulnerable to common network exploits, find all available shares on the network readable by the current user, search for sensitive files on target file shares, enumerate local groups/users for some machines, and more. Check out PowerView&#8217;s _[README.md][13]_ for a complete command listing, and _Get-Help_ for flags and examples for almost all functions.

PowerShell is an extremely effective attack platform for offensive operations, and is an essential tool for penetration testers and red teamers alike. And we&#8217;ve just scratched the surface with current projects- as more attack toolsets are ported to the language we’ll begin to see what&#8217;s truly possible with PowerShell from an attacker perspective. There’s a lot the offensive and professional PowerShell communities can learn from each other, and hopefully a collaborative bridge can be built between our two groups.

[1]: https://www.trustedsec.com/august-2010/powershell_omfg/
[2]: http://www.pentest-standard.org/index.php/Post_Exploitation
[3]: https://twitter.com/mattifestation
[4]: http://www.exploit-monday.com/2012/08/Why-I-Choose-PowerShell.html
[5]: http://www.exploit-monday.com/2014/04/powerworm-analysis.html
[6]: https://www.veil-framework.com/veil-powerview/
[7]: https://twitter.com/mubix
[8]: https://github.com/mubix/netview
[9]: http://msdn.microsoft.com/en-us/library/windows/desktop/bb525382(v=vs.85).aspx
[10]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa370669(v=vs.85).aspx
[11]: http://msdn.microsoft.com/en-us/library/windows/desktop/bb525387(v=vs.85).aspx
[12]: http://www.exploit-monday.com/2012/05/accessing-native-windows-api-in.html
[13]: https://github.com/Veil-Framework/Veil-PowerView/blob/master/README.md