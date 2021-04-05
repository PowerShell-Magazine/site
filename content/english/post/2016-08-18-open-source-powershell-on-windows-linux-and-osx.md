---
title: Open source PowerShell on Windows, Linux, and OS X!
author: PowerShellMagazine
type: post
date: 2016-08-18T15:25:48+00:00
url: /2016/08/18/open-source-powershell-on-windows-linux-and-osx/
views:
  - 34543
post_views_count:
  - 7598
categories:
  - News
  - PowerShell Linux
tags:
  - News
  - Linux

---
_This article was co-authored by [Bartek Bielawski][1] and [Ben Gelens][2]._

Windows PowerShell is a powerful tool, but it always had one very serious limitation: it was possible to run it only on Windows. There had been several attempts to change that, including projects like Pash. However, without support from PowerShell Team these projects had very little chance of catching up with a platform that was growing with each release of PowerShell on Windows.

Almost two years ago in November 2014 when it was announced that .NET Core will become open source project available for multiple platforms including Linux. Many of the PowerShell enthusiasts started to ask the question: when will the same happen to the Windows PowerShell. [Finally, the day has come&#8211;August 18, at 11:14 EST!][3] The time picked for that announcement may look odd at first, but it’s not a coincidence. It’s a way to celebrate anniversary of the public release of Windows PowerShell version 1, on 14 November, 2006. PowerShell is not only open source now, it’s also available for multiple platforms, including several Linux distributions. You can clone [repository][4] and build it on your own, you can fork it and change it and last but certainly not least – you can fix it yourself! Pull requests will be accepted, with details on how to contribute to this project described directly in documentation provided on project’s GitHub page.

### PowerShell on Linux

As you can see this announcement is huge – not only you can freely read and change the code of the PowerShell itself, you can build it and run it almost anywhere. If you want to do just that, you have several packages prepared for different operating systems, including Windows, Linux, and OS X. Packages come with instructions, but these are relatively simple. It all boils down to installing dependencies and the PowerShell package, or just the package itself (if it contains information about dependencies). Installation of CentOS 7.1 requires running single command:

```shell
sudo yum install powershell-6.0.0_alpha.9-1.el7.centos.x86_64.rpm
```

This package has two dependencies (important to know if you want to install PowerShell on the machine that can’t download packages from the external repositories):

```shell
PS /home/bielawb&gt; rpm -qR powershell
libunwind
libicu
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rpmlib(CompressedFileNames) <= 3.0.4-1
```


Once the PowerShell is installed, you should be able to call it, and start exploring it.

![](/images/psl1.png)

### What works and what doesn’t work

What can you expect in Alpha release of PowerShell on Linux? Anything that is related to the core PowerShell functionality works as expected. We have whole group of the object-related commands, we have commands to import, export, and convert multiple file formats. Modules are available and we can create our own modules using PowerShell syntax.

![](/images/psl2.png)

Language constructs work just fine, including configuration (requires PowerShell DSC for Linux to be installed on the same box) and classes. The only exception to this rule is PowerShell Workflow. We can test our code with Pester, we have PSReadLine  to make our life easier. We can create instances of .NET Core classes and behavior is similar to the one we have on Windows.

![](/images/psl3.png)

People like me who expect certain Linux commands to be served as aliases pointing to cmdlet that serve similar purpose should be prepared for a little surprise. I know it took me a good few minutes before I figured out that the result of “ls” is different, because PowerShell doesn’t hide the built-in command with an alias. There are two possible solutions to this problem: define the alias yourself, or use other aliases (e.g. “dir” or “gci” for Get-ChildItem). 

![](/images/psl4.png)

And of course Linux file systems are case sensitive which PowerShell respects:

![](/images/psl5.png)

Another surprise is a result of presence of PSReadLine module. On Linux, iPSReadLine is using emacs mode by default (so certain key bindings are different than the one used on Windows, others have more Linux-like behavior). Both aliases and PSReadLine configuration can be adjusted using familiar technique – creating PowerShell profile:

```powershell
PS /home/bielawb&gt; $PROFILE
/home/bielawb/.config/powershell/Microsoft.PowerShell_profile.ps1
```


PowerShell on Linux still have some missing pieces. First of all, most of the cmdlets that depend strongly on the Windows APIs or full version of .NET Framework are not available. For obvious reasons most of the providers do not exist in PowerShell on Linux. Perfect example of that is registry. Another thing that is not working at the time of writing this article is PowerShell jobs. Also CIM cmdlets are absent at this point in time.

### PowerShell in action

There are probably a lot of scenarios where PowerShell (even in the current, not fully-baked state) can be useful on Linux. Let me name a few of them.

First of all, there are at least two types of objects on Linux that will behave in a way similar to their behavior on Windows: objects in the file system and processes running on our node. The only difference is a result of differences between systems: existing properties, case in the names of files and folders in the file system and more. But that doesn’t change the fact that reading both disk and list of the processes we will get objects back: objects that we can easily sort, group, filter, or format. For example, to display list of processes that use the most CPU, we can use the same line that would give us that information on Windows:

```powershell
Get-Process |
>> Sort-Object -Property CPU -Descending |
>> Select-Object -First 5
```


It’s important to remember that “sort” exists as a Linux command, so we need to use a full name of the cmdlet rather that its alias. Another example, related to the file system: we can get a list of the files in the current directory and pick the one that were changed most recently:

```powershell
PS /home/bielawb> Get-ChildItem |                     
>> Sort-Object -Property LastWriteTime |              
>> Select-Object -Last 1 -Property Name, LastWriteTime
```


We can also turn Linux configuration files into complex objects. Perfect example: grouping existing users based on their shell:

```shell
ipcsv /etc/passwd -Delimiter ':' -Header Name, Pwd, UID, GUID, Info, Home, Shell |
>> Group-Object Shell
```


![](/images/psl6.png)

As you can see – files don’t have to be in any known format, as long as we can pretend they are or structure them in a way that PowerShell understands.

Here is an example of poor-man’s ISE: tmux with vim running in upper pane, and PowerShell running in the lower pane.

![](/images/psl7.png)

### Build your own PowerShell

If you want to start building PowerShell on your own you have two options: install PowerShell from package first and then use tools provided by PowerShell Team in the GitHub repository, or follow the step-by-step instructions and build PowerShell from bash. If you select former method all you have to do is to clone the repository, set your location to newly created folder, import module designed to aid you in the build process and run two commands: one to prepare prerequisites and one to start actual build process:

```shell
git clone --recursive https://github.com/PowerShell/PowerShell.git
cd PowerShell
Import-Module ./build.psm1
Start-PSBootstrap
Start-PSBuild
```


Once the build process is finished, you can launch newly created executable – PowerShell will inform you about location of the finished version.

Manual process is not complicated either (assuming you have prerequisites installed). First, you have to compile libpsl-native Library:

```shell
pushd src/libpsl-native
cmake -DCMAKE_BUILD_TYPE=Debug .
make -j
make test
popd
```


And then, you just need to run “dotnet restore” in the root folder of repository and “dotnet build &#8211;configuration Linux” in the src/powershell-unix folder. But if you just want to use PowerShell on Linux – packages are your best choice. They have very few dependencies and can run on very limited version of Linux.

### PowerShell Remoting over SSH

Another exciting capability I’ve been waiting for is PowerShell remoting over SSH! With PowerShell 6.0.0-alpha.9, all core requirements to make this possible have been introduced.

For Linux end we just need to be sure to install the SSH server Daemon (yum install -y openssh-server if not already installed) and client (yum install -y openssh-client. We should edit the /etc/ssh/sshd_config file to include PowerShell as a subsystem using your favorite text editing tool (e.g. vi, nano).

![](/images/psl8.png)

The rest of the settings are OK by default. These are the required ones:

* PasswordAuthentication yes

* RSAAuthentication yes (optional for RSA key authentication)

* PubkeyAuthentication yes (optional for RSA key authentication)

Then restart the SSH Daemon by typing:

```shell
systemctl restart sshd
```

 We can now remote over SSH. Let’s try it by setting up a remoting session to the localhost first.>

You can use New-PSSession with the -HostName and -UserName parameters. Optionally you could make use of a key file for authentication using the -KeyPath parameter and specifying the path to the key file (this would save you from typing your password interactively). Currently there is no support for PSCredential authentication but this will be added later.

![](/images/psl9.png)

As you can see in the screenshot, you will be prompted to give your consent for connecting to this unknown computer. This will happen only once. Next we have the session available and we can use Invoke-Command with it:

![](/images/psl10.png)

And of course, enter it:

![](/images/psl11.png)

PowerShell is also available on Windows which means we can also remote to a Windows machine over SSH from a Linux machine and vice versa.

To enable this on Windows, we need to install PowerShell for Windows package and also install Win32-OpenSSH (download the latest release here: https://github.com/PowerShell/Win32-OpenSSH/releases). Install OpenSSH to C:\Program Files\OpenSSH using the install instructions found here: https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH. Add C:\Program Files\OpenSSH to the System Path environment variable and you are all set.

![](/images/psl12.png)

If things don’t work for you, your best bet for starting troubleshooting is to stop the SSH Daemon server (“systemctl stop sshd” on Linux or “Stop-Service sshd” on Windows) and run it interactively.

![](/images/psl13.png)

In this example, I intentionally mistyped my password to show the daemon’s output.

### Rapid community adoption

Toolmakers are adopting PowerShell on Linux rapidly. For example, ISESteroids 2.6.1.0 surfaced today and ships with compatibility checkers that help identify code issues that would prevent PowerShell code from running smoothly on Linux (powertheshell.com).

![](/images/psl14.jpg)

### Summary

 was waiting for this day for a very long time. No words can describe how excited I am about the news. I probably won’t be able to fix bugs, or improve documentation (but I’m sure many of you will be). However, I can do many other&#8211;previously impossible or hard to achieve, tasks. I can configure my Linux account to use PowerShell as a primary shell. I can use familiar tools when managing Linux. I can analyze files on the Linux box and make their content behave like a real object. And last but certainly not least: We have finally gotten PowerShell as an OS-agnostic, cross-platform (Windows, Linux, and OS X) automation and configuration tool/framework!

[1]: https://twitter.com/bielawb
[2]: https://twitter.com/bgelens
[3]: https://blogs.msdn.microsoft.com/powershell/2016/08/18/powershell-on-linux-and-open-source-2/
[4]: https://github.com/powershell/powershell