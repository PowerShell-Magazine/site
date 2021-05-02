---
title: Exploring and installing DSC resources published in PowerShell Gallery
author: Ravikanth C
type: post
date: 2014-12-15T17:00:39+00:00
url: /2014/12/15/exploring-and-installing-dsc-resources-published-in-powershell-gallery/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
**Note:** This article applies to Windows Management Framework (WMF) 5.0 Preview release only.

Windows PowerShell team made a formal announcement of [PowerShell Gallery][1] availability (limited preview) last month. The PowerShell Gallery is the place where people can publish their PowerShell modules and DSC resources. People who are interested in using those modules or DSC resources can explore and install them from the PowerShell Gallery directly. This is made possible using a module called [PowerShellGet][2] in Windows Management Framework (WMF) 5.0 Preview release.

I published my [DSC resource modules][3] to the PowerShell Gallery. So, in this article, I will show you how to find the DSC resources in these modules and install them.

There are two different ways you can explore PowerShell Gallery for DSC resources.

### Using Find-Module

The _Find-Module_ cmdlet is a part of the _PowerShellGet_ module in WMF 5.0 Preview release. Using this cmdlet, we can find PowerShell modules and DSC resources published in the PowerShell Gallery.

<pre class="brush: powershell; title: ; notranslate" title="">Find-Module -Includes DscResource -Verbose
</pre>

The possible valid arguments for the -Includes parameter include _DscResource_, _Cmdlet_, and _Function_. By specifying _DscResource_, we specify the intent that we are interested in modules that include DSC resources.

![](/images/findmodule-1024x221.png)

As you see in the above output, we see five modules containing DSC resource. It includes the three resource modules I had published. The other way of using the _Find-Module_ cmdlet is to use tags.

At the time of this writing, note that there are only five resource modules listed here whereas there are more than 50 DSC resource modules in the PowerShell Gallery. This is a known issue.

<pre class="brush: powershell; title: ; notranslate" title="">Find-Module -Tag DSC
</pre>

This will list all modules that carry a &#8220;DSC&#8221; tag that was added while publishing the module. This is not a fool-proof method. If the author did not add this tag during publish phase, it won&#8217;t appear in this list even when the module contains DSC resources.

### Using Find-DscResource

The _PSDesiredStateConfiguration_ module in Windows PowerShell 5.0 Preview has a new cmdlet called _Find-DscResource_. Don&#8217;t confuse this with _Get-DscResource_. The _Get-DscResource_ gets all DSC resources available on the local system whereas _Find-DscResource_ gets you DSC resources from the PowerShell Gallery or any other repository specified using the _-Repository_ parameter.

![](/images/findresource.png)

We can filter this down to a specific module by using the _-ModuleName_ parameter.

![](/images/findbymomdule.png)

Also, if you need only a specific resource, you can get that using the _-Name_ parameter.

![](/images/findbyname.png)

At the time of writing, _-Name_ parameter does not support wildcards.

You can use _-Tag_ property to search for specific tags used while publishing the resource module.

### Installing DSC resource modules

The _Install-Module_ cmdlet can be used to install DSC resource modules from the gallery.

![](/images/install-1024x244.png)

When you use the _Install-Module_ cmdlet with just the _-Name_ parameter, the default install location for the module will be &#8216;_C:\Program Files\WindowsPowerShell\Modules_&#8216;. So, this requires administrative privileges. However, if you want to install the module in the current user context without administrator privileges, you can do so using the _-Scope_ parameter by setting it to _CurrentUser_.

![](/images/local-1024x290.png)

Once you understand how to use these cmdlets, you can point them towards a [local repository too][4].

[1]: https://www.powershellgallery.com
[2]: https://www.powershellgallery.com/pages/GettingStarted
[3]: https://github.com/rchaganti/DSCResources
[4]: http://learn-powershell.net/2014/04/11/setting-up-a-nuget-feed-for-use-with-oneget/