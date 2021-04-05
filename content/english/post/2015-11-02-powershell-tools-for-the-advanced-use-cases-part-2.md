---
title: PowerShell tools for the advanced use cases – part 2
author: "David O'Brien"
type: post
date: 2015-11-02T17:00:35+00:00
url: /2015/11/02/powershell-tools-for-the-advanced-use-cases-part-2/
views:
  - 17603
post_views_count:
  - 3367
categories:
  - DevOps
  - How To
  - Pester
tags:
  - DevOps
  - How To
  - Pester

---
### Posts in this series

  1. [PowerShell Tools for the Advanced Use Cases, part 1][1]
  2. PowerShell tools for the advanced use cases – part 2 (this article)
  3. [PowerShell tools for the advanced use cases – part 3][2]

### Testing

In all teams that follow a “DevOps” mindset there is also the concept of CI (Continuous Integration) or even CD (Continuous Deployment). A part of these concepts is “testing” and validation.

“Is the environment I am going to build in / deploy to ready for deployment?”, “Is what I deployed what I expect it to be?”. The latter is not meant to be “Is what I deployed working the way I expect it to work?”, this would be functional or integration testing.

Today, I would like to introduce something some people call “Technical Verification Tests” (TVT) or “making sure stuff is the way you want it to be” and environment verification. We will first start with the latter.

### Pester your environment

Are you on PowerShell v5? Cool, use PowerShell to install the PowerShell Testing Framework “Pester” onto your machine.

Or, if you’re not on PowerShell 5 yet, go get the module from [here][3], or check the module out on the Gallery itself [here][4].

![](/images/advtools21.png)

```powershell
Install-Module -Name Pester -Repository PSGallery -Scope CurrentUser –Verbose
```


This command will get the PowerShell module “Pester” from the PSGallery (<http://www.powershellgallery.com)> and installs it into your current user’s profile. If you want to check, have a look at “C:\Users\$env:USERNAME\Documents\WindowsPowerShell\Modules”.

Pester is, most of the times, advertised for testing your PowerShell functions, modules or DSC resources. People also mention TDD or Test Driven Development in those cases. I have never been able to get my brain to work “TDD style”, so I won’t get any further into this here than just saying “you first write your tests, initially they all fail, because there’s no actual code, and then you write your code and make your tests pass”. My brain works the other way round.

### Pester environment verification

Environment verification tests are for example run as the first step in a deployment pipeline from a CI/CD server (like Jenkins, Teamcity, Go.CD or maybe even nested Azure Automation runbooks) and check if the environment is ready to be deployed in to before we even start deploying anything. This makes a lot of sense, because you might know what requirements you have for your deployment, but while progressing through environments (Dev, System Test (ST), SIT (System Integration Test), PreProd, Prod) you might not know in which state an environment is in or if a recent change might has broken some of your requirements. So making sure that from a pure deployment perspective you’re good to go is quite valuable.

How would tests like this look like in Pester?

It is easy to set up the skeleton for these tests. With just a couple of lines (on a system with Pester on it) we can create a test skeleton in Pester and fill it with tests like this:

Some very common things to test for are for example “access”; “can I access a remote system?”, “does it listen on specific ports?” or “is there enough free disk space.

Luckily these are all things that are easily written in PowerShell.

```powershell
param (
   [Parameter(mandatory=$false)]
   [string]$server = 'localhost'
)
Describe "environment_verification" {
   It "Tests Port 80" {
      (Test-NetConnection -ComputerName $server -Port 80).TcpTestSucceeded | Should Be 'True'
   }
   It "Tests Port 5986" {
      (Test-NetConnection -ComputerName $server -Port 5986).TcpTestSucceeded | Should Be 'True'
   }
   It "Server's C:\ partition should have minimum 5GB of free disk space" {
      (Get-WmiObject -Class win32_logicaldisk -ComputerName $server -Filter "DeviceID = 'C:'").FreeSpace/1GB | Should BeGreaterThan 5
   }
}
```


### Pester TVT

Technical verification tests (TVT) are usually whatever you want them to be. In my opinion they make sense for things you developed yourself or for processes you don’t really trust. I don’t necessarily go and test if a Windows Feature is really successfully installed if I have used PowerShell DSC to install it, because I trust that its own internal checks are enough.

However, I might want to check custom code that might got used to configure a remote service and where I know that this doesn’t always work (for whatever reason). I hope this makes sense.

In this example I am going to check the IIS configuration that has been given to me from a third party and that I have to work on.

```powershell
Describe "TVT" {
   It "checks if IIS is up and running" {
      (Get-Service -Name W3SVC).Status | Should Be "Running"
   }
   It "checks if IIS anonymous authentication is disabled on Default Web Site" {
      (Get-WebConfigurationProperty -Filter '/system.webServer/security/authentication/anonymousAuthentication' -Name Enabled -PSPath 'IIS:\\' -Location 'Default Web Site').Value | Should Be "false"
   }
   It "checks if IIS Windows authentication is enabled on Default Web Site" {
      (Get-WebConfigurationProperty -Filter '/system.webServer/security/authentication/WindowsAuthentication' -Name Enabled -PSPath 'IIS:\\' -Location 'Default Web Site').Value | Should Be "true"
   }
}
```


That’s all nice, but how are we going to run all this?

### Pester integration into a CD pipeline

Obviously it isn’t the greatest idea to run these tests manually, we want these tests to run before and after every deployment and get the test results. Continuous Integration servers like TeamCity can help with this.

Integrating Pester into a CI pipeline is as straight forward as it gets. The Pester [Github repository][5] even explains how it’s done.

![](/images/advtools22.png)

I saved the first snippet of this article as env_check.ps1, pass it to the Invoke-Pester cmdlet and use Pester’s capability of outputting the test results in the standard NUnitXml format which can then be read and interpreted by TeamCity. TeamCity can then display the test results in the build log for everybody with access to it.

![](/images/advtools23.png)

![](/images/advtools24.png)

Integrating the TVT would work the same way.

### xPlatform Testing

Well, this is all nice and easy if you’re working in an all Windows environment or you can make sure that you can always execute on a Windows machine, but, let’s say this is not the case.

Most CI / CD, some (wrongly?) call them DevOps environments, are not homogenous, meaning pure Windows or pure Linux, they are all pretty much heterogenous and in those cases, it sometimes pains me to say, the platform of choice to control deployments from is Linux.

What are our options in those scenarios?

  * Use Pester still as our testing framework for Windows, because that’s what we know, but we would need some other framework that can trigger Pester from *nix or
  * Use one framework that can test the *nix and Windows platform, because it isn’t a good practise to split this task up

### Serverspec

A testing framework that can do this would be [serverspec][6]. This is written in ruby and based on [rspec][7], a behaviour driven development framework, like Pester, or probably, to be fair, the other way around.

For the sake of this article I will only do a high level introduction to get you started with serverspec.

I am running serverspec during development from my MacBook against Windows environments. Serverspec uses WinRM to communicate to Windows, more specifically it uses the [ruby WinRM module][8]. It can easily use SSL secure communication for this.

My spec_helper.rb file, which configures serverspec, has the following content:

```ruby
require 'serverspec'
require 'winrm'

set :backend, :winrm
user = ‘user’
pass = ‘password’
endpoint = "https://#{ENV['TARGET_HOST']}:5986/wsman"
winrm = ::WinRM::WinRMWebService.new(endpoint, :ssl, :user =&gt; user, :pass =&gt; pass, :basic_auth_only =&gt; true, :no_ssl_peer_verification =&gt; true)
winrm.set_timeout 300 # 5 minutes max timeout for any operation
Specinfra.configuration.winrm = winrm
```

Please note that I am disabling the SSL certificate verification in my test environment.

A typical serverspec file could look like this:

```ruby
require 'spec_helper'
describe port(5985) do
it { should be_listening }
end
describe command('(Get-Process -Name explorer).ProcessName') do
its(:stdout) { should match "explorer" }
end

describe group("Administrators") do
it { should be_exist }
end

describe service("WinRM") do
it { should be_enabled }
it { should be_running }
end
```

The output of it looks similar to Pester’s and can of course also easily be re-used in a CI server.

![](/images/advtools25.png)

### Conclusion

This introduction to infrastructure testing has hopefully shown you the value of investing time in writing these tests.

To conclude, it makes sense testing your assumptions about an environment before deploying into it in order to find issues before even changing anything, i.e. installing software or making changes. They can also be used to check if your Ansible playbooks, Puppet Manifests or Chef cookbooks properly apply the changes, if you don’t trust them in the first place.

[1]: /2015/10/12/powershell-tools-for-the-advanced-use-cases-part-1/ "PowerShell Tools for the Advanced Use Cases, part 1"
[2]: /2015/11/23/powershell-tools-for-the-advanced-use-cases-part-3/
[3]: https://github.com/pester/Pester
[4]: https://www.powershellgallery.com/packages/Pester/
[5]: https://github.com/pester/Pester/wiki/Showing-Test-Results-in-TeamCity
[6]: http://serverspec.org/
[7]: http://rspec.info/
[8]: https://github.com/WinRb/WinRM