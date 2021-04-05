---
title: '#PSConfEU Agenda as a PowerShell Drive using #SHiPS'
author: Ravikanth C
type: post
date: 2018-04-03T16:24:41+00:00
url: /2018/04/03/psconfeu-agenda-as-a-powershell-drive-using-ships/
views:
  - 10420
post_views_count:
  - 4786
categories:
  - PSConfEU
  - PSConf
  - SHiPS
  - Module Spotlight
tags:
  - Conferences
  - Modules
  - SHiPS

---
The [SHiPS][1] module has several use cases with structured data. I have written a few proof-of-concept modules using SHiPS to understand how it works and try out different design patterns.

One of my sessions at [PowerShell Conference EU 2018][2] is around using SHiPS. In the process of creating different demos for this session, I started implementing PS drives for several different things. One such module I created enables the ability to browse PowerShell Conference EU 2018 agenda as a PowerShell drive. I have an initial draft of this module at <https://github.com/rchaganti/PSConfDrive>.

### How to install the module?

Since this is still a very early version of the module, I have not published it yet on the PowerShell Gallery and you need to download the [zip archive][3] of the GitHub repository and extract it to a folder represented by `$env:PSModulePath`. You will require the SHiPS module as well. This can be downloaded from the PowerShell Gallery.

```powershell
Install-Module -Name SHiPS -Force
```


The following commands will load the modules and map a PS drive.

```powershell
Import-Module SHiPS -Force
Import-Module PSConfDrive -Force
New-PSDrive -Name PSConfEU -PSProvider SHiPS -Root psconfdrive#psconfeu
```


Here is how you can use this PS drive for exploring the conference agenda.

![](/images/psconfeuagendadrive.gif)

Once again, this is a POC only and the design still needs to be and can be optimized. If you plan to attend PSConfEU 2018, come to my session on SHiPS to understand how to use the module and choose the right design pattern for your modules.

[1]: https://github.com/powershell/SHiPS
[2]: http://psconf.eu
[3]: https://github.com/rchaganti/PSConfDrive/archive/master.zip