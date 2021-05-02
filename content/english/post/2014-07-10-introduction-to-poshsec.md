---
title: Introduction to PoshSec
author: Matt Johnson
type: post
date: 2014-07-10T16:00:16+00:00
url: /2014/07/10/introduction-to-poshsec/
categories:
  - Infosec
  - Security
  - Module Spotlight
  - PoshSec
tags:
  - infosec
  - security
  - Modules
  - PoshSec

---
Security has always been a passion of mine. It was a few short years ago that I was sitting down having a friendly chat with Will Steele. Like me, Will had a passion for security, specially for the [filtre de confidentialité pc][1] method but also had a passion for PowerShell. There were quite a few times that Will and I discussed how there was a lack of security-related PowerShell modules. At this time the both of us were looking to get involved in a project and a light went off. We decided that we would create what would become PoshSec.

It was around that time that Will unfortunately passed away after a long battle with cancer. I decided then to take the PoshSec torch and run with it. I worked with a few amazing minds on the defensive side of the security industry and worked to craft the project goals and release our first module.

### PoshSec’s Goals

The goal of PoshSec has always been to help system administrators, security analysts and many others have a PowerShell-based way to manage the security on their system. With this in mind, there are a few overall goals that PoshSec tries to achieve with each release going forward:

  1. Provide an avenue to secure, audit, and baseline systems
  2. Rely less and less on items that are not built into Windows by default
  3. Move the PowerShell community forward into the security space

### PoshSec

PoshSec from its inception has provided several areas of coverage for managing, auditing, and baselining systems. Originally, PoshSec focused on the CSIS Top 20 controls and provided features that lined up with the various quick win items that the Top 20 list. The Top 20 list can be found at <http://csis.org/publication/twenty-important-controls-effective-cyber-defense-and-fisma-compliance>. PoshSec has grown since that time and cover areas including forensics and network base lining as well. Currently there are 63 cmdlets/functions in the PoshSec module. We provide coverage for the following areas:

  * Account Monitoring & Control
  * Authorized Devices
  * Forensics
  * Log Management
  * Network Baseline
  * Software Management
  * Utility Functions

![](/images/poshsec1.png)

Let’s go into some of the cmdlets we provide.

### Account Monitoring and Control

When you first look at the items that help with account monitoring and control, you may look at them and say “Hey! The AD module does that already.” I would completely agree. However, the difference is what we provide doesn’t require the AD module or the Active Directory Web service to be installed to function. We utilize the ADSI interface and some .NET classes to provide this functionality. Following one of our stated goal to make things easier for people to use PowerShell to gather this information. This is a huge win for people trying to audit their systems for compliance and for trying to button up the items on the Top 20 control list.

### Log Management

An important area for information security professionals as well as system administrators is logs. Logs will always be important and being able to get the information available to the person who needs it is crucial. We current support gathering information about DNS logs and IIS logs. The 1.5 release due out this fall will have additional log types and functionalities that are supported.

### Software Management

As any good system administrator or analyst can tell you, knowing what is installed on your systems is vital. One of the features we provide is to display the current drivers that are installed on the system. The big win here that we give is the ability to baseline the system to see if anything has changed. The Get-SecDrivers cmdlet allows you to export a CLIXML file with nothing more than specifying the –Baseline switch on the end of the parameter. Below is a screenshot of Get-SecDrivers.

![](/images/poshsec2.png)

### PoshSec Framework

The PoshSec Framework, which will be highlighted in an article later this week by Ben Ten (@ben0xa). The PoshSec PowerShell module is fully integrated with the PoshSec Framework and that takes the overall PoshSec project to a whole new level.

### The PoshSec Community

An old school technology provides direct access to the development team via IRC.  You can find the PoshSec team in the #PoshSec room on Freenode. Additionally, you can find quite a few of the security-related PowerShell developers in the #pssec room on Freenode.

### Where to get PoshSec

PoshSec is currently in version 1.0. You can grab a copy at <http://github.com/poshsec>. We are in a release cadence of twice a year with the next release (version 1.5) coming around in the fall around the infosec cons DerbyCon and GrrCON. We are always welcoming contribution, bug submissions, and feature requests via our GitHub page.

[1]: https://vista-protect.com/fr/produit/filtres-confidentialite-moniteurs/