---
title: Working with PowerShell DSC for Linux, part 1
author: Bartek Bielawski
type: post
date: 2015-02-22T00:15:33+00:00
url: /2015/02/21/working-with-powershell-dsc-for-linux-part-1/
views:
  - 7241
post_views_count:
  - 1379
categories:
  - Linux
  - OMI
  - PowerShell DSC
tags:
  - PowerShell DSC
  - Linux
  - OMI
---
In the <a href="/2015/02/23/working-with-powershell-dsc-for-linux-part-1/" target="_blank">first part</a> of this series, we covered the basis&#8211;how to get Desired State Configuration to do what the name promises: keep desired state consistent. Now that we are sure we will get the expected behavior it&#8217;s time to look into the resources offered in the CTP version of PowerShell DSC for Linux.

The real work (getting, testing and setting system state) in PowerShell DSC for Linux is performed by Python scripts called from C++ based MI providers, with one script per each of the resources. All of these scripts can be found in _$OMI_HOME/lib/Scripts_. Each script defines three main functions (in the similar fashion PowerShell resources do): _Get_Marshall_, _Test_Marshall_ and _Set_Marshall_ plus number of helper functions that &#8220;get the job done&#8221;. Parameters of main functions reflect the schema defined in the MOF document. Because of that design, it&#8217;s not difficult to understand how each of the resources work (as long as you can read Python).

We will start our journey with the resource that is probably the most complex: _nxService_. The complexity is a result of two factors. First of all, Linux has several ways to control daemon&#8217;s configuration. Tooling is not consistent, and even the same Linux distribution can have two different methods to control it. A perfect example is CentOS: CentOS 6 was using _init_, CentOS 7 is using _systemd_. Team responsible for Linux DSC was aware of this and that&#8217;s why in the _nxService_ settings we can decide which controller we want to use. It means that when we define the configuration for a given node, we need to know which method is used, and pick the correct option for that setting. There are three options for controller type. Third controller, not mentioned yet, is _upstart_.

Second reason for resource complexity is related to the way script tests status of services. When you are using text-based parsing of the command output, it is very important to be sure you are using correct locale. Otherwise the actual output and the expected output may not match even though your service is in expected state. Unfortunately, this problem was not identified. Simple example: _init_ controller is using _chkconfig_ command to tell if a given service is enabled. It does that by parsing output from _chkconfig &#8211;list ServiceName_. A fragment of DSC script responsible for test:

```shell
if runlevel_tokens[1] == "on":
        return True
    else:
        return False
```


Actual output from this command on Linux with Polish locale and httpd enabled:

```shell
# chkconfig --list httpd
httpd           0:wyłączone     1:wyłączone     2:włączone      3:włączone      4:włączone      5:włączone      6:wyłączone
```


As you can see, strings do not match. Match will never happen, DSC will forever try to fix something that isn&#8217;t broken any more. To fix this script you would have to enforce locale instead of assuming them. To emulate this we can change environment for a moment and run command in en_US locale:

```shell
# LANG=en_US.UTF8 chkconfig --list httpd
httpd           0:off   1:off   2:on    3:on    4:on    5:on    6:off
```


That works fine for _chkconfig_, but fails for _service_ command:

```shell
# LANG=en_US.UTF8 service httpd status
httpd (pid  1680) jest uruchomiony...
```


Command used to check service current state (_/sbin/service_) is actually a script that we can read and we can try to figure out why it fails. Script calls _env_ command to run service related scripts in clean environment. With clean environment language settings are not transferred to actual script that controls service&#8217;s final state. That&#8217;s bad news. Good news is that reading the content of _/sbin/service_ should be enough to realize that this script is just general wrapper/gateway for scripts kept in _/etc/init.d_ folder. In other words, instead of calling wrapper, we can call script responsible for a given service directly:

```powershell
LANG="en_US.UTF-8" /etc/init.d/httpd status
httpd (pid  1680) is running...
```


That was first option to fix it: modify the code so that it runs actual script, not wrapper. Alternatively, I could take advantage of the fact that service command returns not only localized string, but also exit code that is 0 when service is running, 1 when it&#8217;s not recognized, and 3 when it is stopped. Unfortunately, this change would only fix a problem of status not being properly reported. To keep changes as small as possible I ended up with modifying part that is calling commands and the way service-related script is being called.

Implementing change in Python requires basic knowledge about this language. I had to modify definition of a function _Process_ that was used to call any executable by adding code that would modify environment in which the command should run:

```python
def Process(params):
    enEnv = os.environ.copy()
    enEnv["LANG"] = "en_US.UTF8"
    process = subprocess.Popen(params, env=enEnv, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    (process_stdout, process_stderr) = process.communicate()
    return (process_stdout, process_stderr, process.returncode)
```

That change, plus changing the paths to executables used (scripts in _/etc/init.d_ instead of _/sbin/service_) did the trick. You can find related commits <a href="https://github.com/bielawb/WPSDSCLinux/compare/MSFTOSSMgmt:master...bielawb:25a4e03" target="_blank">here</a>.

Example configuration that uses nxService resource to make sure that Apache is running on a given node:

```powershell
nxService Apache {
    Name = 'httpd'
    Controller = 'init'
    State = 'Running'
    Enabled = $true
}
```


When we use this resource, we have to remember to make sure that we use appropriate controller and the name of service is correct. It&#8217;s complex resource to write, but simple resource to use.