---
title: Introduction to the PoshSec Framework
author: Ben Ten
type: post
date: 2014-07-11T16:00:50+00:00
url: /2014/07/11/introduction-to-the-poshsec-framework/
categories:
  - InfoSec
  - Security
  - Module Spotlight
  - PoshSec
tags:
  - InfoSec
  - Security
  - Modules
  - PoshSec

---
In March of 2013 I had the desire to create an open source Security Information and Event Management system, also known as a SIEM. I had wanted to create something for small businesses or businesses with the “low-to-no” budget for security. I figured it would take a couple of years to get something ready to push out but I knew I had to start somewhere. I had this great idea to write and bake everything in the SIEM and just make it free to the public to use. I realized that it may not take off and that I may abandon the project as I had done with several other projects in the past. Having written applications that spanned multiple different languages, engines, and industry verticals, I realized that this would be a project that I could really enjoy developing and maintaining.

### PoshSec

In June of 2013, I attended a conference called BSides Detroit. At this conference I attended a talk that was presented by <a href="https://twitter.com/mwjcomputing" target="_blank">Matt Johnson</a>. Matt&#8217;s talk was about a project he was working on called PoshSec. Matt recently submitted an <a href="http://104.131.21.239/2014/07/10/introduction-to-poshsec/" target="_blank">article</a> on the history of PoshSec that I encourage you to read.

PoshSec was short for PowerShell Security. Matt talked about how they were using PowerShell to improve the security posture of organizations with a series of cmdlets and scripts. I loved what he was doing and I was really excited about the project. I realized that what I was attempting to create and what Matt was working on could be combined into a new project.

I talked with Matt and we discussed one of the biggest entry points in using PowerShell is that, if you are not a script writer or a developer, it can be a bit intimidating. There are also a series of steps that you need to do before you can run your first script. I realized that what I could do is create a bridge for those who are comfortable with a graphical interface while still giving them the power that comes from a PowerShell script or cmdlet.

In this article, I will be giving you a brief overview of the PoshSec Framework highlighting some of the features benefits and functionality. It is my hope that this tool will prove useful to you and your team in your environment.

### The PoshSec Framework is Born

After I met with Matt, I went back and scrapped my open source SIEM. Instead of having to bake everything inside of my project I would instead create a project that would leverage the power of PowerShell scripts and functions and provide a graphical interface for displaying that information. Don&#8217;t get me wrong, there is still a love for the command line interface. I just know that some members of my team are not that fluent in scripting languages but I wanted to enable them to utilize PowerShell.

The PoshSec Framework (psf) is more than a SIEM. It&#8217;s more than just a security tool. It ties in many System Administration tasks and allows you to interact with the interface and information through the use of custom PowerShell variables. Yes, you can build form elements within PowerShell, but I wanted to make this extremely easy for those who didn&#8217;t want to have to build an entire interface for a simple task.

A video of an early version of psf showing you the basic commands and functions can be viewed below:

{{< youtube bG1W_ridadY >}}

### The First Time Utility

In the first few versions I released, I realized people had a hard time getting their environments set up correctly to use PowerShell, much less psf. Therefore I decided to create a First Time utility in psf. When you run psf for the first time it will go through and do the following steps for you to ensure that your environment is configured properly to use PowerShell:

  1.  Verify that the settings are correct for psf to function properly.
  2. Do an initial download of the PoshSec PowerShell modules from GitHub.
  3. Unblock any files that were downloaded.
  4. Perform the Update-Help command.
  5. Set the Execution-Policy to RemoteSigned.

You have the option of skipping the First Time utility, but this has significantly helped people get PowerShell configured properly so they can begin using scripts and cmdlets as well as psf. I have created a video of the early version of the first time utility that did not have the GitHub download in it. The latest version of psf has full GitHub integration. You can watch that video below.

{{< youtube ivtYyoHeIWw >}}

### Integrations

#### **Syslog**

One of the features I had to incorporate was writing any alerts that are generated in psf to a Syslog server. Therefore in the logging tab in settings, you can not only write all output and alerts to a file but you can send the alerts to a Syslog server. This means that your PowerShell script can now generate an alert that will go to your existing Syslog server without any heavy lifting in your script.

![](/images/image00311.gif)

If you set up the Alert Log to use Syslog whenever you call the $PSAlert.Add() method it will not only show up in the Alerts tab but it will also send that same alert to the Syslog you specified.

### GitHub Integration

The other feature that I added to psf was direct integration with GitHub. There are so many wonderful community-based PowerShell projects. Especially in the security sector. To make it easy to get those projects, I have enabled a direct API call to GitHub to download the project, unpack it, and make it available in the runspace. You only need to put the URL to the project and select which branch you would like to use; psf does all of the rest of the work.** **

![](/images/image00411.gif)

### The Back-End

I created psf to have its own runspace and individual pipelines for each script, function, or command that is run. This means you can run several scripts, commands, or functions at the same time without having to launch a new instance of PowerShell. You can manage those scripts in the scripts tab. I don&#8217;t just send a command to the powershell.exe process. I create an entire runspace and process everything in the System.Management.Automation.dll file. This creates a powerful environment for managing scripts and cmdlets.

### The Layout

The initial view of psf has many elements available. You have your domains listed, any systems that were discovered in your scan, a listing of scripts, the listing of PowerShell commands, aliases, and functions, and a section for alerts and active scripts. I even have a tab for a PowerShell command line interface so you can still issue commands via a command line. You can also launch a separate instance of the command prompt or PowerShell with buttons on the toolbar.

![](/images/image00511.gif)

### Exposed Elements

There are elements of the interface that are directly exposed so that you can access them from within your PowerShell script or cmdlet. This allows you to directly interact with the psf interface without having to build your own form or interface handlers.

For example, if you wanted to display a MessageBox in PowerShell, you would need to create a function or cmdlet with code that is similar to the example below.

<pre class="brush: powershell; title: ; notranslate" title="">[System.Reflection.Assembly]::LoadWithPartialName(“System.Windows.Forms”)
[Windows.Forms.MessageBox]::Show(“Hello there PowerShell Magazine Readers!”, “PowerShell Magazine Demo”, [Windows.Forms.MessageBoxButtons]::OK, [Windows.Forms.MessageBoxIcon]::Information)
</pre>

However, in psf this is the only thing you need to add to your script or cmdlet:

<pre class="brush: powershell; title: ; notranslate" title="">$PSMessageBox.Show("Hello there PowerShell Magazine Readers!", "PowerShell Magazine Demo")
</pre>

The $PSMessageBox variable is exposed directly through the runspace in psf. You don&#8217;t need to initialize or set up anything. Here is that command run directly in psf on the PowerShell tab.

![](/images/image00611.gif)

Another example is creating a grid view with columns and rows for a PSObject collection. You have the Out-GridView command in PowerShell, but it doesn&#8217;t do much more than display the data. In psf, you can create a new tab in the interface with $PSTab. For example, here is how I would put the output of Get-ChildItem on my c:\psf directory into a grid view and its own tab in psf:

<pre class="brush: powershell; title: ; notranslate" title="">$PSTab.AddObjectGrid($(Get-ChildItem C:\psf\), "PSF Directory")
</pre>

Here is what that looks like in the psf interface. You notice the new “PSF Directory” tab next to Scheduled Scripts.

![](/images/image00711.gif)

![](/images/image00811.gif)

This tab also allows you to export this information as XML, CSV, or raw text. I also expose the computer information listed in the Systems tab with $PSHosts. You can also access the alerts tab with $PSAlerts; as well as create a separate tab just for alerts. There are several other ways you can interact with the interface, but in keeping this article an overview we won&#8217;t cover all of them. I have a few other videos that highlight those features <a href="https://www.youtube.com/watch?v=41lIdbdRHVw" target="_blank">here</a> and <a href="https://www.youtube.com/watch?v=2YlTj7iyYsQ" target="_blank">here</a>.

Scheduling Scripts

One of the ways that we utilize psf in our environment is by scheduling our scripts for base-lining and comparison. This way we can track if any new software has been installed, a new user was added to our domain, or even if a system is opening a new port that it hasn&#8217;t before. I could have used the built in scheduling system in Windows, however, I wanted to keep everything in psf. Therefore I created a scheduling component to schedule your scripts.** **

![](/images/image00911.gif)

The nice thing about scheduling through psf is that you can store all of the command or script parameters in the scheduled job; including the selected hosts from the Systems tab. This enables you to set up specific scripts for a group of servers or set up a script if you want to monitor a single box.  For example, the three scripts you see above do the following:

  1. filechanges.ps1 monitors a file and adds an alert for any changes to a specific file. I use this for one of our internal web applications which has built in logging. When the log updates I get an alert in psf as well as through my Syslog server.
  2. monitor-startupprograms.ps1 monitors all of my servers start up programs. This ensures that I am aware of any changes to programs that are set to run at start up.
  3. monitor-ad-accounts.ps1 monitors my domains for any additions or deletions of any objects. Any changes are set to the alerts as well as my Syslog server.

### There is Much More

As you can see in this brief overview, the goal of psf is to harness the power of PowerShell and make it available to use in a graphical format. This enables more people to begin utilizing amazing scripts that are being written by the PowerShell community without having to be a script writer or developer. My goal for psf is not to replace the PowerShell CLI or even PowerShell ISE. My goal is to provide a tool to enable PowerShell to be more widely adopted. With the scripts and cmldets written by the PoshSec team, and other people like Carlos Perez (@darkoperator), Matt Graeber (@mattifestation), Chris Campbell (@obscuresec), and Will Schroeder (@harmj0y) I wanted to share their experience and security focus with other organizations to strengthen their overall security landscape.

There is more that is being added to psf. It is currently at version 1.0. You can download the source code as well as the binary from the PoshSec GitHub page at [https://GitHub.com/poshsec/poshsecframework][1]. Feel free to log any requests, bugs, or issues on our GitHub page.

[1]: https://github.com/poshsec/poshsecframework