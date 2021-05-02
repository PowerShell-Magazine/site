---
title: 'PowerShell DSC for Linux: Resources'
author: Bartek Bielawski
type: post
date: 2015-07-08T16:00:58+00:00
url: /2015/07/08/powershell-dsc-for-linux-resources/
featured_image: /wp-content/uploads/2012/01/logo-250x250.png
views:
  - 8949
post_views_count:
  - 1920
categories:
  - PowerShell DSC
  - Linux
tags:
  - PowerShell DSC
  - Linux
---
New version of PowerShell DSC for Linux doubled the number of resources available. Initial batch in CTP covered basics: nxUser to create and manage users, nxGroup to define groups and manage group members, nxFile to create and modify files, nxService to start, stop, enable, and disable daemons, and nxScript to cover anything that couldn&#8217;t be done with other resources. We covered these resources in the <a href="/2015/02/21/working-with-powershell-dsc-for-linux-part-2/" target="_blank">previous series</a> and they haven&#8217;t changed much since.

New resources cover installation of packages (nxPackage), injecting single line into existing file (nxFileLine), configuring environment variables (nxEnvironment), working with archived data (nxArchive) and setting up authorized SSH keys (nxSShAuthorizedKeys). In this part of the series we will focus on these newly-added resources.

### nxPackage

We will start with nxPackage. For me personally that is the most important resource from those added in this release. I created <a href="/2015/02/26/working-with-powershell-dsc-for-linux-part-4/" target="_blank">composite resource for previous version</a> to cover this gap. Unlike my composite resource, the one that shipped covers several package managers. It has support for local installation of packages. You just have to specify _FilePath_ to package file. It understands concepts of package groups. You can specify _Arguments_ for installation and define what is expected _ReturnCode_. I&#8217;m a CentOS user. So, I&#8217;ve tested it with _yum_. It worked fine for any scenarios that I&#8217;ve tried. For very basic scenarios when you specify absolute minimum information, the only required property is the _Name_:

```powershell
Configuration packages {
    Import-DscResource -ModuleName nx
    node LinuxNode {
        nxPackage apache {
            Name = 'httpd'
        }
    }
}
```


If you don&#8217;t specify package manager the resource will try to find one for you. That means that if you are not sure or have a mixed environment with distributions that resemble both Debian and RedHat, you can use one configuration and it will &#8220;just work&#8221;. The part of code that does the magic is using _which_ command:

```python
def GetPackageManager():
    ret = None
    # choose default - almost surely one will match.
    for b in ('apt-get', 'zypper', 'yum'):
        code, out = RunGetOutput('which ' + b, False, False)
        if code is 0:
            ret = b
            if ret == 'apt-get':
                ret = 'apt'
            break
    return ret
```


In my opinion that is very elegant solution to the problem of package management diversity between Linux distributions.

If we don&#8217;t want to depend on the resource to find package manager for us or we want to be sure that specific manager is used, we need to specify it in our configuration. If given manager is not present on remote system, we should get an error but that is unfortunately not very verbose. To find actual cause, we would have to use my favorite (and from my experience&#8211;the most effective) debugging technique, run omiserver interactively:

```shell
CalledProcessError.  Error Code is 127
CalledProcessError.  Command string was dpkg-query -W -f='${Description}|${Maintainer}|'Unknown'|${Installed-Size}|${Version}|${Status}
' httpd
CalledProcessError.  Command result was /bin/sh: dpkg-query: command not found
check installed:/bin/sh: dpkg-query: command not found

CalledProcessError.  Error Code is 127
CalledProcessError.  Command string was apt-get  install  --allow-unauthenticated --yes  httpd
CalledProcessError.  Command result was /bin/sh: apt-get: command not found
Failed to Install httpd output for command was: /bin/sh: apt-get: command not found
```

My system doesn&#8217;t have apt-get on it, so error message is expected.

If we want to remove packages, we can do it by specifying value _Absent_ to Ensure property.

### nxFileLine

Another interesting and potentially useful resource is nxFileLine. Unlike nxFile, this one is suitable for any component that have just one configuration file without option to read parts of the configuration from other files stored on disk. In such a case, replacing whole file may not be the best option. What we need is an option to remove certain lines or add lines that we need. Former can be achieved with a _DoesNotContainPattern_ property which accepts regular expressions. Latter is done with _ContainsLine_ property that specifies the line that needs to be appended. You have to be aware that there is no correlation between the two: new line is always added at the end of a file, so if position in the file is important; nxFileLine won&#8217;t help you. Example configuration that makes sure that _sshd_ is not trying to resolve hosts (handy in lab environment) and removes any line that is just a comment:

```powershell
Configuration Lines {
    Import-DscResource -ModuleName nx
    node LinuxNode {
        nxFileLine sshd {
            FilePath = '/etc/ssh/sshd_config'
            DoesNotContainPattern = '^#.*?$|UseDNS yes'
            ContainsLine = 'UseDNS no'
        }
    }
}
```


### nxEnvironment

Another common need on Linux system is configuration of environment variables. Their purpose and behavior is similar to the one we observe on Windows: most of variables are product-specific and are used only by certain applications. There is one environment variable that stands out: _PATH_. The only difference between both systems is separator: Windows is using semicolon where Linux is using colon. One of the resources added in this release (nxEnvironment) is designed to configure these variables, including the _PATH_. Example configuration adds _OMI_HOME_ variable and adds folder with OMI binaries to the _PATH_:

```powershell
Configuration Environment {
    Import-DscResource -ModuleName nx
    node LinuxNode {
        nxEnvironment Normal {
            Name = 'OMI_HOME'
            Value = '/opt/omi'
        }
        nxEnvironment Path {
            Name = 'WhoCares?'
            Path = $true
            Value = '/opt/omi/bin'
        }
    }
}
```


This configuration highlights two differences between &#8220;normal&#8221; variables and the PATH variable. Former use name to specify the name of variable, for _PATH_ it&#8217;s still a key value, but used only to differentiate between several items added to the path. To tell the resource that it&#8217;s a path that we configure we need to provide value _$true_ to Path parameter. Another difference is a way both types are defined on Linux box. _PATH_ is configured in _/etc/profile.d/DSCEnvironment.sh_ by appending each item to existing _PATH_ with colon as a separator. Any other variable definition is added to _/etc/environment_ that is dot-sourced in _DSCEnvironment.sh_. The only problem I&#8217;ve found so far with this resource is related to _Ensure_ property. Specifying _Present_ works fine, but _Absent_ don&#8217;t seem to change anything if variable is defined in another script under _/etc/profile.d_.

### nxArchive

Another resource, nxArchive, should help us with using archived files. All you need to do is specify path to archive and destination for the files. This resource have very similar properties as Archive available for Windows. The difference is checksum usage: we can choose between md5, mtime, and ctime. Another difference is lack of credential support and validate property. Example configuration would use archive _/tmp/source.tgz_ and create _/tmp/target_ folder using it content:

```powershell
Configuration Archive {
    Import-DscResource -ModuleName nx
    node 192.168.7.204 {
        nxArchive tmp {
            SourcePath = '/tmp/source.tgz'
            DestinationPath = '/tmp/target'
            Checksum = 'md5'
        }
    }
}
```


### nxSshAuthorizedKeys

Last resource added in this release should address issue of managing SSH keys for a given user. With nxSshAuthorizedKeys resource we can deliver keys to any number of nodes and be sure that our private key will work seamlessly with all of them. It will take care of proper file structure and permissions on both _.ssh_ folder and _authorized_keys_ file:

```shell
[root@PSMag ~]# ls /home/bielawb/.ssh/ -las
total 4
0 drwx------. 2 bielawb bielawb  28 Jun 14 01:29 .
0 drwx------. 3 bielawb bielawb  90 Jun 14 01:29 ..
4 -rwx------. 1 bielawb bielawb 424 Jun 14 01:29 authorized_keys
```


Something that might require change is rights for _authorized_keys_ files: _x_ is not necessary. It&#8217;s not a big issue: keys will work just fine with either.

There are two odd things about this resource: first of all, _KeyComment_ is marked as a key in the schema. Probably because there was no natural candidate that could be used: same key value may need to be applied to several users, same user can have multiple keys. Other one is more important: even though schema doesn&#8217;t define _UserName_ as required, it is required by the resource. This means that instead of getting errors at compilation phase, we will get them at implementation phase. Example configuration (with actual key removed):

```powershell
Configuration SshKeys {
    Import-DscResource -ModuleName nx
    node LinuxNode {
        nxSshAuthorizedKeys bielawb {
            Key = 'ssh-rsa key description'
            KeyComment = 'How will that work?'
            UserName = 'bielawb'
        }
    }
}
```


Because there isn&#8217;t any strict link between resource key and actual data stored in the file we can easily break existing one by adding single line between comment and actual key data, or by removing comment from the file. If that happens DSC will duplicate the line with a key. It&#8217;s not a big issue, but just something that user should be aware of.

We now know what we can configure currently with PowerShell DSC for Linux. But there is also huge change in how we can achieve that. We will take a look at that in the third and final part of this series.