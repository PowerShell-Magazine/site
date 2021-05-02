---
title: Working with PowerShell DSC for Linux, part 4
author: Bartek Bielawski
type: post
date: 2015-02-26T17:00:57+00:00
url: /2015/02/26/working-with-powershell-dsc-for-linux-part-4/
views:
  - 22704
post_views_count:
  - 2876
categories:
  - PowerShell DSC
  - Linux
  - OMI
tags:
  - PowerShell DSC
  - Linux
  - OMI

---
CTP of Linux DSC that was released last year has only five resources: _nxService_, _nxFile_, _nxUser_, _nxGroup_, and _nxScript_. Things like package management, network configuration, firewall settings, jobs scheduled in crontab don&#8217;t have coverage in resources. Some of these issues can be addressed with _nxFile_, as we discussed in <a target="_blank">part 3</a>. Luckily, with _nxScript_ we can cover the rest.

Resources for scripts on Linux and Windows are very similar. They share same properties and provide similar functionality. The difference is a side effect of how both operating systems are handling scripts and it gives _nxScript_ some benefits that Windows counterpart doesn&#8217;t provide. On *nix systems, file extensions are not as important as they are in Windows. It may help some tools present on Linux to provide functionality (e.g. syntax highlighting in code editor), but system is not using file name extension to tell him what the given files is. Scripts are perfect examples of that behavior.

In Linux, script interpreter is defined in the first line. This descriptor, called &#8220;shebang&#8221;, has following format:

<pre class="brush: bash; title: ; notranslate" title="">#! Interpreter [arguments]
</pre>

Most of the time we simply provide a path to binary that will execute directives from our script. For example, the following line will make sure that our file is considered a Python script by the shell that we are using:

<pre class="brush: bash; title: ; notranslate" title="">#!/usr/bin/python
</pre>

That works fine if we know the path. At times the path may be different than default one. Also it may be different on various systems. To avoid hardcoding the path to interpreter, we can use _env_ command which will resolve the name to first executable present in the _PATH_ variable. For example to mark our script as Perl script we could provide following shebang:

<pre class="brush: bash; title: ; notranslate" title="">#!/usr/bin/env perl
</pre>

Important thing to remember: shebang has to be the first line of our file. To prove that extensions are not important, we will define two simple scripts that use Python syntax (_print_). One with extension that suggests shell script (.sh) and Python shebang, and one with extension that suggests Python script (.py) and without shebang:

```shell
# ./fake.sh
I'm a python script, see?
# ./relly.py
./relly.py: line 1: print: command not found
```


Why is that important? First of all it means that as long as we can depend on the interpreter being present on the Linux system that we configure, we can use any scripting language we want. Even though script we send over the wire is saved with sh extension we don&#8217;t have to think about it, we just need to format shebang properly. But it also means that if resource is not taking shebang into account, our scripts have to be formatted appropriately: with shebang in the very first line. If we use a herestring, then it should not be an issue. But if we define *Script properties using script block syntax, we are doomed to fail. Example configuration:

```powershell
configuration scriptBlock {
    param (
        [String]$ComputerName
    )
    Import-DscResource -ModuleName nx

	node $ComputerName {
    	nxScript mix {
        	GetScript = {
				#!/usr/bin/python
				print "Worked!"
            }
            TestScript = {
                #!/bin/bash
                exit 1
            }
            SetScript = {
                #!/bin/perl
                print "Mixing different langs"
            }
        }
    }
}
```

Even though it looks like we got our code right, it is incorrect. Anything after the brace is already considered a part of our script. If we look at actual scripts created on Linux (if script fails DSC will leave it in _/tmp/*/temp_script.sh_), they all have extra blank line:

```shell
# for file in /tmp/*/temp_script.sh; do  cat -n $file; echo EOF; done

 1
 2  #!/bin/bash
 3  exit 1
 4              EOF
 1
 2  #!/bin/perl
 3  print "Mixing different langs"
 4              EOF
```


To fix this issue and keep script block syntax, we would have to move the first line immediately after the curly brackets. In my opinion, it shouldn&#8217;t be necessary: resource should be smart enough to remove unnecessary white spaces and leave just the part that can be used. To get that behavior I modified _nxScript_ a bit. You can find these changes in my <a href="https://github.com/bielawb/WPSDSCLinux/commit/379be9179af34288595268cbf1e10e1c6bdfc785" target="_blank">fork of Linux DSC</a>. Regardless of your decision &#8211; use official version or my fork &#8211; I would suggest using herestrings for script&#8217;s definition. Even if we don&#8217;t have to be careful with white spaces at the beginning of a file, it&#8217;s a bit easier to keep it clean. On top of that: using string syntax prevents us from thinking that any variable expansion is being done on the client side before configuration is sent to remote node. We can easily verify that using two simple configuration items that are really just &#8220;get&#8221; scripts (test will always return &#8220;True&#8221;):

```powershell
configuration getOnly {
    param (
        [String]$ComputerName,
        [String]$Message
    )
    Import-DscResource -ModuleName nx
    node $ComputerName {
        nxScript fail {
            SetScript = {}
            TestScript = {#!/bin/bash
                exit 0
            }
            GetScript = {#!/bin/bash
                echo Got $Message with ScriptBlock
            }
        }
        nxScript work {
            SetScript = ''
            TestScript = @"
#!/bin/bash
exit 0
"@
            GetScript = @"
#!/bin/bash
echo Got $Message with HereString
"@
        }
    }
}

$get = getOnly -ComputerName 192.168.200.104 -Message Foo
Update-MofDocument -Path $get.FullName
Start-DscConfiguration -CimSession $linTest -Wait -Path $get.DirectoryName
Get-DscConfiguration -CimSession $linTest | ForEach-Object Result

Got with ScriptBlock
Got Foo with HereString
```

If we use string everything is done as expected: variables are expanded in double-quoted string, not expanded in single-quoted string. Using string syntax allows us to easily use format operator and be sure that certain placeholders are being replaced by parameters provided by the user. Using this syntax we can build composite resource for adding or removing packages from Linux box. This is just a sketch &#8211; it assumes we use _yum_ as a package manager and that makes it useful only for certain Linux distributions:

```powershell
configuration nxPackage {
    param (
        [Parameter(Mandatory)]
        [String]$Name,
        [ValidateSet('Present','Absent')]
        [String]$Ensure = 'Present'
    )
    Import-DscResource -ModuleName nx
    nxScript "Package-$Name" {
        GetScript = @'
#!/bin/bash
yum info {0}
'@ -f $Name
        TestScript = @'
#!/bin/bash
case {1} in
    Present)
        yum list installed {0} &gt; /dev/null 2&gt;&1 || exit 1 && exit 0;;
    Absent)
        yum list installed {0} &gt; /dev/null 2&gt;&1 || exit 0 && exit 1;;
esac
'@ -f $Name, $Ensure
        SetScript = @'
#!/bin/bash
case {1} in
    Present)
        yum -y install {0};;
    Absent)
        yum -y erase {0};;
esac
'@ -f $Name, $Ensure
    }
}
```


Once we have this resource defined we can create configuration using it, for example, if we want to install _nano_ and _Apache_ on a server that uses yum as a package manager:

```powershell
configuration PackageList {
    param (
        [String]$ComputerName
    )
    node $ComputerName {
        nxPackage nano {
            Name = 'nano'
        }
        nxPackage httpd {
            Name = 'httpd'
        }
    }
}
```


If we move this composite resource to nx (or any other) module than we need to import that module first in our configuration. Similar to Windows composite resource, to make nxPackage &#8220;proper&#8221; resource we need to create manifest that points to schema.psm1 file with configuration definition. Once it is done we can use it as if it was any other resource. The main difference is visible when we call Get-DscConfiguration. As composite resource we have created is present only on Windows it will be translated to _nxScript_ resources locally and delivered to Linux and we will see _nxScript_ resources results, not nxPackage results.

Another example of action that seems like a perfect fit for script resource is firewall configuration. This is pretty straightforward if iptables is used to manage firewall settings:

```powershell
nxScript AddFirewallRule {
    SetScript = @'
#!/bin/bash
iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport http -j ACCEPT
/etc/init.d/iptables save
'@
    TestScript = @'
#!/bin/bash
iptables -L | grep ^ACCEPT | grep "dpt:http "
exit $?
'@
    GetScript = @'
#!/bin/bash
iptables -L | grep ^ACCEPT | grep "dpt:http "
'@
}
```


This configuration works fine (some error handling though would be more than welcome). But not all distributions are using iptables (at least &#8211; not directly). If we look at CentOS 7 we will noticed that different method (firewalld) is used. We can control firewall configuration using _firewall-cmd_ command, but this command have problems with being called from non-interactive session. Configuration will literally freeze until we kill the process that is running firewall command line utility. It means that following configuration, even though it looks legit, won&#8217;t work in normal circumstances:

```powershell
nxScript AddFirewallRuleFirewalld {
    SetScript = @'
#!/bin/bash
firewall-cmd --permanent --add-service http
firewall-cmd --reload
'@

	TestScript = @'
#!/bin/bash
firewall-cmd --list-service | grep '\bhttp\b'
'@

	GetScript = @'
#!/bin/bash
firewall-cmd --list-service
'@
}
```

The problem is related to SELinux policies and rights that _firewall-cmd_ has. Two workarounds that worked for me: disable SELinux or run omiserver interactively. There is also a solution though.

First we need to reproduce problems, and then find the related error messages in audit.log, and finally create policy based on that &#8220;pattern&#8221;. Once we have policy file we can apply the change to our system:

```shell
ps aux | grep [f]irewall-cmd | awk '{print "pid=" $2 " /var/log/audit/audit.log" }' |
    xargs grep | audit2allow -M firewall
semodule -i firewall.pp
```


With this change implemented we can use _firewall-cmd_ in our configuration without freezing behavior.

Note: Both tools (semodule and audit2allow) are part of separate packages: _policycoreutils_ and _policycoreutils-python_, respectively. If either command doesn&#8217;t work make sure you have installed these packages. It&#8217;s enough to install latter as former is one of its dependencies and will be installed anyway.

And that&#8217;s it in this part of the series. In next one we will cover _nxUser_ and _nxGroup_.