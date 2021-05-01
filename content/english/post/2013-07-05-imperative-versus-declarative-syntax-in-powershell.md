---
title: Imperative versus declarative syntax in PowerShell
author: Ravikanth C
type: post
date: 2013-07-05T16:00:03+00:00
url: /2013/07/05/imperative-versus-declarative-syntax-in-powershell/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
The big news, for the PowerShell enthusiasts, from TechEd NA 2013 was the [Desired State Configuration][1] (DSC) feature announcement. DSC can be used to streamline configuration changes in a datacenter environment and in a consistent manner. This feature is built right into the core OS so that not just PowerShell but other ISVs and management software can also leverage the underlying management model.

If you have heard Jeffrey Snover and Kenneth Hansen speak about this topic at TechEd, they mentioned, quite a few times, [declarative][2] syntax that they have blended with the [imperative][3] nature of Windows PowerShell. For an IT professional with no development background, someone like me for example, this terminology isn&#8217;t very familiar. To appreciate the real benefits of a feature like DSC and the way it is implemented, we need to first understand the differences between declarative and imperative scripting practices. And, that is the essence of this article. Of course, I will use PowerShell to showcase the difference.

Make a note that this article is not about whether declarative syntax is good or bad. Instead, this article is going to introduce what declarative and imperative syntax are.

Let us first look at imperative style of programming/scripting and start with an example to understand this better.

```
Import-Module ServerManager

#Check and install ASP.NET 4.5 feature
If (-not (Get-WindowsFeature "Web-Asp-Net45").Installed) {
    try {
        Add-WindowsFeature Web-Asp-Net45
    }
    catch {
        Write-Error $_
    }
}

#Check and install Web Server Feature
If (-not (Get-WindowsFeature "Web-Server").Installed) {
    try {
        Add-WindowsFeature Web-Server
    }
    catch {
        Write-Error $_
    }
}

#Create a new website
Add-PSSnapin WebAdministration
New-WebSite -Name MyWebSite -Port 80 -HostHeader MyWebSite -PhysicalPath "$env:systemdrive\inetpub\MyWebSite"

#Start the website
Start-WebSite -Name MyWebSite
```

If you look at the above example, we are telling PowerShell **how to perform what we need to perform**. The emphasis here is on how we perform a task and in the process we achieve what we need to. This is called the imperative programming/scripting style and is what we write everyday in PowerShell. We need to explicitly code how to verify the task dependencies and how the exceptions need to be handled. Going back to our example, I am explicitly checking if ASP.Net 4.5 and Web Server features are installed or not and then install them if they are not present.

Enter PowerShell 4.0 and the Desired State Configuration feature. The Windows PowerShell team has chosen to implement declarative programming style for DSC:


    Configuration WebSiteConfig
    {
        Node MyWebServer
        {
            WindowsFeature IIS
            {
                Ensure = "Present"
                Name = "Web-Server"
            }
            WindowsFeature ASP
            {
                Ensure = "Present"
                Name = "Web-Asp-Net45"
            }
    
            Website MyWebSite
            {
                Ensure = "Present"
                Name = "MyWebSite"
                PhysicalPath = "C:\Inetpub\MyWebSite"
                State = "Started"
                Protocol = @("http")
                BindingInfo = @("*:80:")
            }        
    	}
    }
Observe the above example. We are more or less doing the same job&#8211;as in imperative style example&#8211;in this code snippet. That is, checking if ASP.Net 4.5 and Web Server features are installed and then create a website. Within the DSC code snippet, we are specifying **what needs to be done and not how it needs to be done**. In the declarative programming style, we are not concerned about how things are done. We depend on the underlying automation or programming framework to know how to perform a given set of tasks. Of course, there has to be explicit support within the underlying framework to perform these tasks. This is what DSC resources enable. The declarative programming style is more readable and easy to understand.

To summarize, the **imperative syntax** defines **how a task should be performed** while **declarative syntax** describes **what needs to be done**.

This article is not about DSC feature itself but the programming style used to implement DSC. In the future articles, we will describe DSC more in-depth and understand how to use it. Stay tuned.

[1]: http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/MDC-B302
[2]: http://en.wikipedia.org/wiki/Declarative_programming
[3]: http://en.wikipedia.org/wiki/Imperative_programming