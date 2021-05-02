---
title: Writing help for custom DSC resources
author: Ravikanth C
type: post
date: 2014-03-21T17:00:23+00:00
url: /2014/03/21/writing-help-for-custom-dsc-resources/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
Desired State Configuration is the hottest thing out there and everyone is writing custom DSC resources for almost everything. One of the most important aspects of writing custom DSC resources is to write help content on how to use these custom resources. All of us know that custom DSC resources are nothing but PowerShell modules. But, these modules contain a similar set of functions. These functions are _Get-TargetResource_, _Set-TargetResource_, and _Test-TargetResource_. So, writing comment-based help for these functions does not really make sense. More over, these functions are the back end mechanism for performing configuration changes. The real help that is required for a custom DSC resource should be about how to use the declarative syntax of the resource.

The way this can be solved is by using conceptual help topics or the about topic text files. The about help topics are one of the different methods of [writing module help][1] in PowerShell.

Let&#8217;s look at an example. One of the DSC resources I&#8217;d written was the _HostsFile_ resource. I had written the about topic for this module and here is the folder structure for this DSC resource.

![](/images/dsctree2.png)

If you look at the folder structure and file names, you will see that I have localized the help content. For example, I have written only the English help content. So, I&#8217;d created a folder named _en-US_ which is the UI culture on my system. Under this folder, I placed the help content for the _HostsFile_ resource as an about text file and named it about\_DSCResource\_hostsfile.help.txt_. Make a note of the naming convention used. It is mandatory that the file name must end with .help.txt. I recommend that you use the naming convention that makes it easy to identify the DSC resource help content. For example, I am using _about\_DSCResource\_<resourcename>.help.txt_. Also, remember that the file must be saved using _UTF-8_ encoding.

Once you have the above criteria met, you can simply look at the about help content using the command _Get-Help about_*_.

![](/images/helpabout1.png)

Here is the partial content of my <em>about_</em> text file for <em>HostsFile</em> DSC resource.

![](/images/aboutfile1.png)

By following this approach, you should be able to add help content on how to use your custom DSC resources.


[1]: http://msdn.microsoft.com/en-us/library/dd878343(v=vs.85).aspx