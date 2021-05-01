---
title: Security in a Changing World
author: Lee Holmes
type: post
date: 2014-07-07T16:00:37+00:00
url: /2014/07/07/security-in-a-changing-world/
categories:
  - Security
  - InfoSec
tags:
  - Security
  - InfoSec

---
If you surveyed the landscape of computer security a decade ago, you would find a world completely different than the one we live in today.

In the early 2000s, our industry was still reeling from the impact of Code Red, Nimda, and SQL Slammer. While some operating systems had network firewalls, few enabled them by default. It was a world where server misconfigurations and accidentally exposing services to the internet reigned supreme.

In that world, network security forms the battle ground. Slap an Enterprise Grade™ firewall on your network edge, add a demilitarized zone (DMZ), and call it a day. Attackers back then focused on full frontal assault: port scanning, service compromise, buffer overflows, denial of service, and more.

That world still exists, and by no means can we consider that battle won. Our industry has matured, however, and we’ve made significant strides in protecting ourselves from this type of attack. As our network security has improved and those pickings have become slim, attackers have moved on to more creative attacks on the most critical components of the network: those that manage it. With just a little bit of social engineering or phishing, most networks are quick to fall.

When it comes to social engineering or phishing, the words simply don’t effectively illustrate the danger. If you haven’t seen the brutal efficiency of these attacks before, I invite you to watch <a href="https://twitter.com/kevinmitnick" title="Kevin Mitnick on Twitter" target="_blank">Kevin Mitnick</a> and <a href="https://twitter.com/HackingDave" title="Dave Kennedy on Twitter" target="_blank">Dave Kennedy</a> demolish a corporate fortress with a single phone call:

{{< youtube DB6ywr9fngU >}}

Four minutes after their target first picks up the phone, a couple of random dudes from Las Vegas are able to evade hundreds of thousands of dollars of security equipment and run arbitrary code on the company’s corporate network. In front of an audience of a couple hundred hackers, to boot.

Now, once an attacker tricks a user to run arbitrary code on their machine, [it’s not that user’s machine anymore][1]. And worse: whatever that code decides to do has the same power as if the user were to do it by hand. It doesn’t matter if they’re tricked into running a Java applet, malicious PDF document, batch file, or PowerShell script – the permissions and capabilities of that compromised user account are now completely open to abuse.

If you’re uncomfortable with what a compromised administrator could do to your cloud or corporate network, it would be well worth your time applying the principles of Just Enough Administration (JEA). Jeffrey Snover gave an excellent deep dive on the topic at TechEd North America 2014: “_[_A Windows PowerShell Toolkit to Secure a Post-Snowden World_][2]_”.

<iframe style="height: 352px; width: 625px;" src="http://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DCIM-B362/player?h=352&w=625&format=html5" width="300" height="150" frameborder="0" scrolling="no" allowfullscreen="allowfullscreen"></iframe>

In this new world of “assume breach”, PowerShell plays a few interesting roles.

Do you remember the exhilaration you felt the first time you replaced a hundred-line VBScript tool with a PowerShell one-liner? While PowerShell can’t do anything you couldn’t already do with a custom-written C++ or VBScript application – it’s just so much more fun! This same realization is slowly growing within the offensive and defensive security community. When attackers are already running arbitrary code on a network, they’re beginning to realize that writing a registry dumper in PowerShell is infinitely easier than doing it in assembly language or C++.  Or that querying Active Directory with the AD cmdlets is way more fun than programming the LDAP queries by hand. While they could accomplish the task through other (more complicated) means, job satisfaction is just as important to an attacker as it is to you and I.

This uptick of PowerShell usage in the security community isn’t restricted to attackers, of course. As you’ll see this week, PowerShell forms an incredible platform for security analytics and network defense. Whether you’re reverse engineering malware, troubleshooting a suspicious system, or enforcing strict administrative boundaries on your network, PowerShell is truly making it easier than ever to adapt to an ever-changing world.

[1]: http://technet.microsoft.com/library/cc722487.aspx
[2]: http://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DCIM-B362