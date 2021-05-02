---
title: Working with PowerShell DSC for Linux, part 5
author: Bartek Bielawski
type: post
date: 2015-02-27T17:00:20+00:00
url: /2015/02/27/working-with-powershell-dsc-for-linux-part-5/
views:
  - 9287
post_views_count:
  - 1632
categories:
  - PowerShell DSC
  - OMI
  - Linux
tags:
  - PowerShell DSC
  - linux
  - OMI
---
Managing Linux can become a challenge if you don&#8217;t have some way of distributing credentials. There are several options: joining Active Directory domain and using shared credentials for both Linux and Windows, using other LDAP implementation to authenticate or distribute users/groups using Configuration Management tools. In the final part of Linux DSC series we will take a look at last two resources &#8211; _nxUser_ and _nxGroup_ &#8211; that should help us manage user and groups on Linux systems. We will also take a look at few debugging techniques that should help you identify a reason why given configuration fails.

Adding a user on a Linux with _nxUser_ resource is relatively simple task. It&#8217;s usually enough to specify username, password, and a home folder. The password is a tricky part: we have to provide it in salted hash form. I couldn&#8217;t find any reasonable solution for Windows to create such hashes, so I ended up creating a simple script on the Linux box that would generate the salted hash of selected password for me:

```python
#!/usr/bin/python
from crypt import crypt
from optparse import OptionParser


parser = OptionParser(
    description='Creates salted hash using specified plain-text password and salt'
)

parser.add_option(
    '-p',
    '--password',
    help = 'Plain text password'
)

parser.add_option(
    '-s',
    '--salt',
    help = 'Salt that will be used',
    default = 'PowerShellMag'
)

(options, args) = parser.parse_args()

if not options.password:
    parser.error('Password is required!')

print crypt(
    options.password,
    "$6$%s$" % options.salt
)
```

There are at least few methods to copy this file over to Linux box: scp, sftp, or even nxFile resource. Once the script is there, we can call it. Again: there are few options available. I decided to use Posh-Ssh module <a href="/2014/07/03/posh-ssh-open-source-ssh-powershell-module/" target="_blank">described in PowerShell Magazine a while ago</a>:

```powershell
$linuxSession = New-SSHSession -ComputerName 192.168.200.104 -Credential $bielawbCred -AcceptKey $true

$salt = Invoke-SSHCommand -SSHSession $linuxSession -Command 'bin/passgen.py -p P@ssw0rd' | ForEach-Object {
    $_.Output -replace "`n"
}
```

Our configuration should take at least two parameters: name of the computer and a salted hash, which we will generate at runtime. Of course, we would have to change salt or we will get the same salted hash every time. Configuration that would create Web Admin account on target node:

```powershell
configuration LinuxUser {
    param (
        [String]$ComputerName,
        [String]$PasswordSalt
    )

	Import-DscResource -ModuleName nx

	node $ComputerName {
    	nxUser WebAdmin {
        	Ensure = 'Present'
        	UserName = 'webadmin'
        	FullName = 'Web Admin'
        	Password = $PasswordSalt
    	}
	}
}
```

In this case we have used _nxUser_ to create a user, but nothing prevents us from updating users using the same resource. For example, we may want to update password for a user using content of _/etc/shadow_ on the reference machine. First we need to run SSH command as the root to read hash for a given user:

```powershell
$rootSession = New-SSHSession -ComputerName 192.168.200.104 -Credential $root -AcceptKey $true
$salt = Invoke-SSHCommand -SSHSession $rootSession -Command 'grep bielawb /etc/shadow | cut -d: -f2' | ForEach-Object {
    $_.Output -replace "`n"
}
```


Next, we add _nxUser_ to our configuration script. In this case we need a few parameters: name of the node, name of the user, and user&#8217;s password:

```powershell
configuration UpdatePassword {
param (
    [String]$ComputerName,
    [String]$UserName,
    [String]$PasswordSalt
)
    Import-DscResource -ModuleName nx
    # (...)
    node $ComputerName {
        nxUser "$UserName-Update" {
            UserName = $UserName
            Password = $PasswordSalt
        }
    }
}
```


As a final step we have to apply the configuration to selected nodes. After this is done we can use the same password on all nodes where this configuration was applied.

Creating a user is only part of the picture. The group membership is usually more important as it is used to grant users certain rights on the Linux box. With _nxGroup_ resource we can create groups, specify group members, request preferred group ID, and select users that should be added or removed from a given group. Example configuration that creates group _web_ with _bielawb_ and _webadmin_ as a members and makes sure that _bielawb_ is a member of the group _wheel_ but not group _apache_ and at the same time _webadmin_ is a member of the group _apache_ but not a member of the group _wheel_:

```powershell
configuration LinuxGroup {
    param (
        [String]$ComputerName
    )

	Import-DscResource -ModuleName nx

	node $ComputerName {
    	nxUser WebAdmin {
        	Ensure = 'Present'
        	UserName = 'webadmin'
    	}

		nxGroup WebAdmins {
    		Ensure = 'Present'
    		GroupName = 'web'
    		Members = 'bielawb', 'webadmin'
    		DependsOn = '[nxUser]WebAdmin'
		}

		nxGroup Apache {
    		GroupName = 'apache'
    		MembersToInclude = 'webadmin'
    		MembersToExclude = 'bielawb'
    		DependsOn = '[nxUser]WebAdmin'
		}

		nxGroup Wheel {
    		GroupName = 'wheel'
    		MembersToInclude = 'bielawb'
    		MembersToExclude = 'webadmin'
    		DependsOn = '[nxUser]WebAdmin'
		}
	}
}
```

We covered basics and all of the resources, it&#8217;s time for a few hints that may save you from hours of banging your head against the desk.

First of all: you can skip the _Verbose_ parameter on _Start-DscConfiguration_. Based on my experience it doesn&#8217;t provide any useful information. You have to look for hints on the other side of the wire, on the Linux box that you are talking to. First of all &#8211; you can read logs. There are two logs that may give you some ideas in case your configuration doesn&#8217;t work as expected. Both can be find in _$OMI_HOME/var/log_. There is a log for omiserver (_omiserver.log_) and separate one for dsc (_dsc.log_). If amount of information in the omiserver log is not sufficient, we can increase the logging level (_&#8211;loglevel_, default is 2, max value is 5) or log also http traffic (_&#8211;httptrace_).

DSC depends on OMI. OMI usually runs in the background as a service. I strongly recommend to run it interactively while debugging: most of the Python code used in the resources doesn&#8217;t have any error handling. When we run omiserver interactively, we can see errors/exceptions that bubble up and based on these errors find out what failed:

```shell
Set
Traceback (most recent call last):
  File "/opt/omi-1.0.8/lib/Scripts/nxFile.py", line 33, in Set_Marshall
    retval = Set(DestinationPath, SourcePath, Ensure, Type, Force, Contents, Checksum, Recurse, Links, Owner, Group, Mode)
  File "/opt/omi-1.0.8/lib/Scripts/nxFile.py", line 428, in Set
    if SetFile(DestinationPath, SourcePath, fc) == False:
  File "/opt/omi-1.0.8/lib/Scripts/nxFile.py", line 368, in SetFile
    shutil.copyfile(SourcePath, DestinationPath)
  File "/usr/lib64/python2.6/shutil.py", line 50, in copyfile
    with open(src, 'rb') as fsrc:
IOError: [Errno 2] No such file or directory: u'/not/there'
```


With this extra information spotting the problem should be easier.

Linux DSC was announced last year and published as a CTP. As any other CTP, it&#8217;s not perfect. However, it would help to see how it changes in the same location that it was initially published, on GitHub. Without this it may seem that the project was dropped. Lack of the official fixes leaves anybody who identifies a problem with two options. First options is to fix the problem. Second options is to use unofficial version of the product. I chose the first one. I hope this series convinced you that my attempt was good enough, and encourage you to give Linux DSC a try. And that concludes this series about Linux DSC &#8220;in action&#8221;.

