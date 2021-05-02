---
title: Working with PowerShell DSC for Linux, part 2
author: Bartek Bielawski
type: post
date: 2015-02-23T17:00:34+00:00
url: /2015/02/23/working-with-powershell-dsc-for-linux-part-2/
views:
  - 9741
post_views_count:
  - 1862
categories:
  - PowerShell DSC
  - Linux
  - OMI
tags:
  - PowerShell DSC
  - Linux
  - OMI
---
PowerShell DSC for Linux caught my attention immediately after it was announced. Being able to control configuration of Linux box with pure PowerShell code is huge for anybody who is interested in cross-platform solutions build in PowerShell. PowerShell DSC depends on OMI and that makes it even more interesting. I fell in love with WMI at first sight and OMI is open source implementation of WMI for (almost) any device/operating system.

If you are also interested in this subject you&#8217;ve probably read Ravi&#8217;s article on <a href="/2014/05/21/installing-and-configuring-dsc-for-linux/" target="_blank">Installing and configuring DSC for Linux</a>. I&#8217;ve also read it and use it almost every time I was installing DSC for Linux. I would either check prerequisites, or read details about DSC installation procedure. But if you thought that DSC operates the way it should once it is installed, you should think again.

This series of articles will describe all the things that I discovered while playing with Linux DSC &#8212; issues I had to address, problems I had to solve. This should be a nice supplement to the modified version of <a href="https://github.com/bielawb/WPSDSCLinux" target="_blank">Linux DSC CTP</a> that you can find under my GitHub account. Problems I&#8217;ve had identified didn&#8217;t surprise me. Issues are expected side effects of working with CTP version of any product.

### Consistency not invoked

If you continue to use the machine that you&#8217;ve installed DSC on and you occasionally change the context to the root account you will soon notice that the number of mails sent to this user is growing:

```shell
2 (Cron Daemon)         Mon Jan 19 09:00  27/989   "Cron <root@ITPro> /opt/omi-1.0.8/bin/ConsistencyInvoker"
    3 (Cron Daemon)         Mon Jan 19 09:30  27/989   "Cron <root@ITPro> /opt/omi-1.0.8/bin/ConsistencyInvoker"
 U  4 (Cron Daemon)         Mon Jan 19 10:00  27/988   "Cron <root@ITPro> /opt/omi-1.0.8/bin/ConsistencyInvoker"
```


It&#8217;s your cron daemon begging for mercy. Yes, you did push your DSC configuration. You either configured it to fix itself or not &#8211; that is not important so much as the fact that once configuration is pushed you will get following entry in _crontab_:

```shell
# cat /etc/crontab | grep omi
*/30 * * * * root /opt/omi-1.0.8/bin/ConsistencyInvoker
```


It reflects LCM configuration, so if you increase time interval between refreshes, _crontab_ configuration will follow. But _crontab_ complains this file is missing, and quick look reveals that this complaint is justified:

```shell
# cat /etc/crontab | grep omi | awk '{print $NF }' | xargs ls -l
ls: cannot access /opt/omi-1.0.8/bin/ConsistencyInvoker: No such file or directory
```


My initial thought was that probably executable exists on my disk but was simply overlooked in installation process. But _find_ revealed only few folders. One of them looked promising though:

```shell
# find / -name ConsistencyInvoker -print
/root/build/WPSDSCLinux/dsc/LCM/dsc/engine/ConsistencyInvoker
...
# ls -l /root/build/WPSDSCLinux/dsc/LCM/dsc/engine/ConsistencyInvoker
total 8
-rw-r--r--. 1 root root 3751 Jan 20 20:57 ConsistencyInvoker.c
-rw-r--r--. 1 root root  297 Jan 20 20:57 GNUmakefile
```


We have C code and compiler instructions so we can check where the executable file did go. It appears that name of created executable matches the one referenced in _crontab_, but it&#8217;s all lower case. Compiled program is also not copied to the final location. But copying the file to the correct location is just a partial solution. It will prevent _crontab_ from sending you messages, but system state will remain untouched.

### It works! Except it doesn't

The problem with consistency program is that even though executable is compiled without any errors it doesn't address configuration drift. Initially with my limited C/C++ skills I walked around the problem by using remote CIM calls. I even created function that would simplify invoking required CIM method:

```powershell
function Invoke-DscOperation {
	param (
        [Parameter(
            Mandatory
        )]
        [Microsoft.Management.Infrastructure.CimSession]$CimSession,
        [ValidateRange(0,3)]
        [Int]$Flag = 1
    )
    $methodParam = @{
        CimSession = $CimSession
        Namespace = 'root/Microsoft/Windows/DesiredStateConfiguration'
        ClassName = 'MSFT_DSCLocalConfigurationManager'
        MethodName = 'PerformRequiredConfigurationChecks'
        Arguments = @{
            Flags = [uint32]$Flag
        }
    }

	Invoke-CimMethod @methodParam
}
```

Recently I decided to give it a shot and try to fix the problem on the Linux side. I have used &#8220;debugging&#8221; method that is typical for people without proper skills: &#8220;printf debugging&#8221;. And because program is written in C it was not a concept, it was literally _printf_ that revealed what the problem is and gave me a hint on how to fix it. My first step was simpler though. I just wanted to clarify if _ConsinstencyInvoker_ results in an error and doesn&#8217;t report it, or it works fine but fails to do its job. To confirm that I&#8217;ve used construct that I miss in PowerShell occasionally:

```powershell
# consistencyinvoker || echo failed!
failed!
```


If you never used this construct: it will run second command only if the first one have non-zero return code. Output assured me that running _ConsinstencyInvoker_ results in the silent error, so I modified the code a bit to figure out what went wrong. First of all I had to add reference to _stdio.h_ so that I could use _prinft_ function. The only other change I made was located in the last method call. I printed the error message to the console, and that already gave me a good hint where the problem is:

```shell
# consistencyinvoker
Error: The target namespace does not exist.
```


Now I just needed to figure out which namespace this application is trying to use. Walking &#8220;back&#8221; in the code from the method call lead me to variable with the name that didn&#8217;t leave any room for doubt: _DSCENGINE_NAMESPACE_. One more _printf_ and I was no longer surprised that _ConsinstencyInvoker_ doesn&#8217;t work:

```shell
# consistencyinvoker
Error: The target namespace does not exist.
Namespace: /dsc
```


In the end I just had to modify one line to get proper namespace: _#define_ instruction responsible for value of _DSCENGINE_NAMESPACE_ constant. You can find actual change in above mentioned <a href="https://github.com/bielawb/WPSDSCLinux/commit/3e94c0149cc4f3e1bde7b84b14d9eb3c385e69a4" target="_blank">GitHub repository</a>

Once that was fixed I got executable that was working correctly. And with Local Configuration Manager (LCM) set to _ApplyAndAutocorrect_, I could easily test if configuration drift is being fixed automatically.

### Install that just works

After I&#8217;ve fixed _ConsinstencyInvoker_ there was just one element remaining. Compiled executable was not copied/moved to the location where _crontab_ was expecting to find it. As you probably remember from Ravi&#8217;s post, installation of DSC contains two steps: _make_ and _make reg_.

As registration of DSC providers requires similar rights to the one required to copy programs to system-wide path it makes perfect sense to add appropriate _cp_ command to one of the make files. Analyzing make files starting from main located in root of source quickly revealed location that was perfect for command that was overlooked in original installation. The file in question contains several copy commands. Target for this part of make file (deploydsc) matches exactly our intent.

But we can&#8217;t hardcode source and destination path. After analyzing variables defined in various rules files I have identified two that should point to correct source and destination:

```shell
# cat LCM/GNUmakefile | grep consistencyinvoker
        cp  $(BINDIR)/consistencyinvoker $(CONFIG_BINDIR)/ConsistencyInvoker
```


Again, you can find committed change in my <a href="https://github.com/bielawb/WPSDSCLinux/commit/f382a959604d0831366e0c0d707a6df23f1414fe" target="_blank">GitHub repository</a>. After applying both changes installation of the package and providers was pretty straight forward. With both changes in place we are ready to install Linux DSC without problems I had identified. But installation/configuration problems of the core DSC engine are not the only problems I have observed (and fixed). After all: DSC is only as good as the providers it offers. In the next part of the series we will look into the problems that may show up when we work with nxService provider.