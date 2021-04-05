---
title: Containerizing a web application
author: Jan Egil Ring
type: post
date: 2017-11-20T17:00:50+00:00
url: /2017/11/20/containerizing-a-web-application/
views:
  - 21165
post_views_count:
  - 9065
categories:
  - Containers
  - How To
tags:
  - Containers
  - How To

---
In this article, we will look at different options for containerizing a web application.

We will go through the following deployment scenarios, going from traditional options to cloud services and containers:

  * Deploy to local machine
  * Deploy to an Infrastructure as a Service (IaaS) VM
  * Deploy to a Platform as a Service (PaaS) website
  * Deploy to a container running Windows Server Core
  * Deploy to a container running Nano Server

Our example application is the [PowerShell Universal Dashboard][1] – a web application built on ASP .NET Core, PowerShell Core, ReactJS, and a handful of JavaScript libraries.

This means it can support cross-platform deployments, running on Windows, Linux, and macOS.

That is already a very flexible range of options, but we can get even more options by using containers.

The PowerShell Universal Dashboard PowerShell module allows for creation of web-based dashboards. The client- and server-side code for the dashboard is authored completely with PowerShell. Charts, monitors, tables and grids can easily be created with the cmdlets included in the module. The module is a cross-platform module and will run anywhere PowerShell Core can run.

![](/images/contain1.png)

### Deployment options

** **

#### Deploy to local machine

Let us first look at how to install and setup the application locally on a client machine.

Install the UniversalDashboard module from the PowerShell Gallery using PowerShellGet:

Install-Module UniversalDashboard

When the module is installed you will have access to a number of new commands:

![](/images/contain2.png)

Start-UDDashboard is the main command for starting a new instance of a dashboard.

For the example usage of the other commands, I would encourage you to have a look at the [Components section][2] of the beforementioned demo website.

The example dashboard I am going to use for this article is one from a real-world scenario. During an onboarding process, there was a need to gather some data (an employee number) from an external company. The link was to an instance of PowerShell Universal Dashboard, which would take the input from the external user and send it as a parameter to an [Azure Automation webhook][3]. The webhook would start an Azure Automation runbook, which would register the data in the internal system.

The dashboard for this scenario is available in [this Gist][4].

When the UniversalDashboard PowerShell module is installed, we can simply run the dashboard.ps1 script to start the dashboard. Here is a screenshot from performing this in Visual Studio Code:

![](/images/contain3.png)

At this point, the website should be up and running. We can test by navigating to http://localhost/register/123

![](/images/contain4.png)

In this example, we are also taking input from the URL. 123 can be any number (in the example scenario, a service request ID), and will be available as a parameter variable in the New-UDInput command.

At this point, we have the application up and running on a local computer as a PowerShell module. For production use, we want to run this on a server platform.

As stated in the [product documentation][5], the PowerShell Universal Dashboard can be hosted in Azure or IIS:

To host a dashboard in Azure or IIS, you will need to deploy the entire module to your site or WebApp. In the root module directory, place your dashboard.ps1. You need to specify the -Wait parameter on Start-UDDashboard so that the script blocks and waits for requests in Azure or IIS. Specifying the port isn&#8217;t necessary because Azure and IIS use dynamic port tunneling.

_IIS requires that the_ [ASP.NET Core module][6] _is installed._** **

#### Deploy to an Infrastructure as a Service (IaaS) VM running Internet Information Services (IIS)

The other option would be to simply run the website in IIS on a Windows Server which can run anywhere: on-premises, a public cloud, or a hosting provider.

In this example, we are using a virtual machine running in Azure based on the Azure Marketplace  [Windows Server Datacenter, version 1709][7] image.

Step 1 – Install the web server role: Install-WindowsFeature -Name Web-Server

Step 2 – Install the .NET Core Windows Server Hosting bundle as described [here][8] (needed since the PowerShell Universal Dashboard is built on .NET Core and not the built-in .NET Desktop edition).

Step 3 – Copy the UniversalDashboard PowerShell module to the path where the website is running, for example C:\inetpub\wwwroot if the Default Web Site is leveraged.

Step 4 – Copy the dashboard.ps1 and license.lic files to the same directory:

![](/images/contain5.png)

That\`s it – at this point the website should be up and running.

#### Deploy to a Platform as a Service (PaaS) website

The route for the mentioned real-world scenario was to host the module in a public cloud service, in this case Azure App Service:

![](/images/contain6.png)

There are many different options for deploying a web application to an instance of Azure App Service. Microsoft has provided some examples for us to use:

  1. [Create a web app with deployment from GitHub][9]
  2. [Create a web app with continuous deployment from GitHub][10]
  3. [Create a web app and deploy code with FTP][11]
  4. [Create a web app and deploy code from a local Git repository][12]
  5. [Create a web app and deploy code to a staging environment][13]

#3 was used to build the demo website for this article, but in a production environment a more appropriate method would be to leverage continuous integration to deploy files to the website based on commits from source code.

We get many benefits from leveraging a PaaS offering such as Azure App Service:

  * No servers to manage (no patching, monitoring, etc)
  * We can add custom domains and SSL certificates as part of the service
  * (bigger VM sizes)
  * Scale in and out (more VM instances)
  * Deploy the application using continuous delivery such as Visual Studio Team Services

There are also many other benefits such as pre-authentication, load balancing and more.

### Containerization

Next, we will look at leveraging containers &#8211; a solution to the problem of how to get software to run reliably when moved from one computing environment to another.

This is a new technology with a promise to change the IT landscape the same way as virtualization did in the early 2000s.

#### Deploy to a container running Windows Server Core

Our first example of containerizing the PowerShell Universal Dashboard will be based on Windows Server Core, version 1709. Since we already have the application up and running on a native operating system, it should be easy in this case to transform it into a container.

The files for the following demos are available [here][14].

In the WindowsServerCoreDemoWebsite, we have the following files:

![](/images/contain7.png)

dashboard.ps1 and license.lic are the same files we used when running the application in a native operating system. These will be referenced in the Dockerfile to be copied into the container image.

In this example, we are using Docker – a container management tool – to build and deploy our demos. From the [documentation][15]:

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using docker build users can create an automated build that executes several command-line instructions in succession.

Let us have a look at the [Dockerfile][16] which defines our Windows Server Core, version 1709 demo website:

![](/images/contain8.png)

We are using an image from Docker Hub – a central repository for Docker images – which is built by Microsoft and have IIS pre-installed. Then we do not have to think about installing and configuring the Web-Server role.

Next, we are downloading and installing the .NET Core Windows Server Hosting bundle like we did when running the application on a native operating system.

PowerShellGet is used to download the UniversalDashboard module.

The last step is to copy the dashboard.ps1 and licence.lic files as well as exposing the port the website is running on.

```powershell
cd "~\Git\PSCommunity\Containers\PowerShell Universal Dashboard"
```

> Note: Remember to switch to Windows Containers before building the docker file (Linux is the default after installing Docker for Windows)

```powershell
docker build WindowsServerCoreDemoWebsite -t psmag:demowebsite --no-cache
docker build NanoDemoWebsite -t psmag:nanodemowebsite --no-cache

#region 1 Windows Server Core
$ContainerID = docker run -d --rm psmag:demowebsite
$ContainerIP = docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" $ContainerID
```

Verify that the website is up and running.

```powershell
Start-Process -FilePath iexplore.exe -ArgumentList http://$ContainerIP/register/123
Start-Process -FilePath chrome.exe -ArgumentList http://$ContainerIP/register/123
```

Optionally, connect to a container instance interactively to inspect the environment. The IIS image have a service monitor as an entrypint, thus we need to override this to get into the container interactively

```powershell
docker run --entrypoint=powershell -it psmag:demowebsite

docker stop $ContainerID

#endregion

#region 2 Nano Server 1709
$ContainerID = docker run -d --rm psmag:nanodemowebsite
$ContainerIP = docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" $ContainerID
```

Verify that the website is up and running.

```powershell
Start-Process -FilePath iexplore.exe -ArgumentList http://$ContainerIP/register/123
Start-Process -FilePath chrome.exe -ArgumentList http://$ContainerIP/register/123
```

Optionally, connect to the container instance interactively to inspect the environment

```powershell
docker exec -ti $ContainerID pwsh #pwsh for Nano/powershell for Server Core
docker stop $ContainerID
#endregion
```

When the Dockerfile is ready, we can user docker.exe to build a container image (line 4-5).

When the image is successfully built, we are ready to test it by starting a container instance using our new image (line 8).

If you have made modifications to dashboard.ps1 or simply want the latest version of the UniversalDashboard module, re-run the docker build command and the image will be updated with any changes.

#### Deploy to a container running Nano Server

Before we try to make our demo application run on Nano Server, there are some [important changes][17] to Nano Server introduced in Windows Server 1709 to be aware of:

Starting with the new feature release of Windows Server, version 1709, Nano Server will be available only as a container base OS image. You must run it as a container in a container host, such as a Server Core installation of Windows Server. Running a container based on Nano Server in the new feature release differs from earlier releases in these ways:

  * _Nano Server has been optimized for .NET Core applications._
  * _Nano Server is even smaller than the Windows Server 2016 version._
  * _PowerShell Core, .NET Core, and WMI are no longer included by default, but you can include PowerShell Core and .NET Core container packages when building your container._
  * _There is no longer a servicing stack included in Nano Server. Microsoft publishes an updated Nano container to Docker Hub that you redeploy._
  * _You troubleshoot the new Nano Container by using Docker._
  * _You can now run Nano containers on IoT Core._

One more thing that is useful to know, but not mentioned in the referenced article, is that IIS is not available in Nano Server 1709.

This means we need to take a different approach to get the application running in a Nano-based container.

PowerShell Universal Dashboard is built on top of [Kestrel][18] &#8211; a cross-platform web server for ASP.NET Core. This means we can simply run Start-UDDashboard from PowerShell Core in Nano Server 1709 to get the web application up and running.

[Rick Strahl][19] has written a great article about [Publishing and Running ASP.NET Core Applications with IIS][20] where he mentions the following:

Kestrel is a .NET Web Server implementation that has been heavily optimized for throughput performance. It&#8217;s fast and functional in getting network requests into your application, but it&#8217;s ‘just’ a raw Web server. It does not include Web management services as a full featured server like IIS does. If you run on Windows you will likely want to run Kestrel behind IIS to gain infrastructure features like port 80/443 forwarding via Host Headers, process lifetime management and certificate management to name a few.

The bottom line for all of this is if you are hosting on Windows you&#8217;ll want to use IIS and the AspNetCoreModule.

Some of the limitations can be overcome by leveraging different options such as a PaaS service or custom reverse proxy to publish the application externally, as well as a container orchestration tool such as Kubernetes for scaling and managing the application.

With these limitations in mind, let us go on and see if we can get this working on the latest version of Nano Server.

As mentioned previously, PowerShell Core has been removed from Nano Server starting with the 1709 release. The first step would be to build a new container image where PowerShell Core is included.

Luckily, the PowerShell team have published [the Dockerfile][21] for the official Docker image for PowerShell Core on GitHub.

In our [custom Dockerfile][22], we are leveraging almost all of the Dockerfile used to build PowerShell Core. We are also using the concept of [multi stage builds][23] to include ASP .NET Core by referencing FROM microsoft/aspnetcore:2.0-nanoserver-1709 in the Dockerfile.

The only code we need to add is to download the UniversalDashboard PowerShell module as well as copy the dashboard.ps1 file as we did when running on Server Core:

![](/images/contain9.png)

When using IIS, licence.lic was automatically read by the application. This is not the case when using the module directly from PowerShell, hence we have added Set-UDLicense to specify the licence inside dashboard.ps1.

Since IIS is not available, we need to add an entry point in the Dockerfile in order to launch the dashboard.ps1 file. This will launch the website by calling Start-UDDashboard.

You might notice that PowerShell Core is called by using pwsh.exe. With the release of PowerShell Core 6.0.0-beta.9 the binary for PowerShell Core was renamed from powershell.exe and powershell on Windows and Linux/Unix/macOS respectively to pwsh.exe and pwsh. [Mark Kraus][24] has written a great [article][25] with more background info about that change.

A final note about Nano Server: On the [Azure Marketplace offering for Windows Server][7], there is an offer called Windows Server Datacenter, version 1709 with Containers. If you deploy an instance of that image, Docker will be pre-installed and you can download the latest official Microsoft image of Nano Server from Docker Hub by running docker pull microsoft/nanoserver:1709.

### Summary

In this article, we have explored various options for hosting a web application in different environments, starting with a native operating system and ending up with a very small container image based on Nano Server.

[1]: https://poshtools.com/powershell-universal-dashboard/
[2]: http://www.poshud.com/Components
[3]: https://docs.microsoft.com/en-us/azure/automation/automation-webhooks
[4]: https://gist.github.com/janegilring/05f916b9852616125030938f94b0f5e4
[5]: https://adamdriscoll.gitbooks.io/powershell-tools-documentation/content/powershell-pro-tools-documentation/universal-dashboard/running-dashboards.html
[6]: https://docs.microsoft.com/en-us/aspnet/core/hosting/aspnet-core-module
[7]: https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.WindowsServer
[8]: https://docs.microsoft.com/en-us/aspnet/core/publishing/iis?tabs=aspnetcore2x
[9]: https://docs.microsoft.com/en-us/azure/app-service/scripts/app-service-powershell-deploy-github?toc=%2fpowershell%2fmodule%2ftoc.json
[10]: https://docs.microsoft.com/en-us/azure/app-service/scripts/app-service-powershell-continuous-deployment-github?toc=%2fpowershell%2fmodule%2ftoc.json
[11]: https://docs.microsoft.com/en-us/azure/app-service/scripts/app-service-powershell-deploy-ftp?toc=%2fpowershell%2fmodule%2ftoc.json
[12]: https://docs.microsoft.com/en-us/azure/app-service/scripts/app-service-powershell-deploy-local-git?toc=%2fpowershell%2fmodule%2ftoc.json
[13]: https://docs.microsoft.com/en-us/azure/app-service/scripts/app-service-powershell-deploy-staging-environment?toc=%2fpowershell%2fmodule%2ftoc.json
[14]: https://github.com/janegilring/PSCommunity/tree/master/Containers/PowerShell%20Universal%20Dashboard
[15]: https://docs.docker.com/engine/reference/builder/
[16]: https://github.com/janegilring/PSCommunity/blob/master/Containers/PowerShell%20Universal%20Dashboard/WindowsServerCoreDemoWebsite/Dockerfile
[17]: https://docs.microsoft.com/en-us/windows-server/get-started/nano-in-semi-annual-channel
[18]: https://github.com/aspnet/KestrelHttpServer
[19]: https://twitter.com/rickstrahl
[20]: https://weblog.west-wind.com/posts/2016/Jun/06/Publishing-and-Running-ASPNET-Core-Applications-with-IIS
[21]: https://github.com/PowerShell/PowerShell/blob/master/docker/release/nanoserver/Dockerfile
[22]: https://github.com/janegilring/PSCommunity/blob/master/Containers/PowerShell%20Universal%20Dashboard/NanoDemoWebsite/Dockerfile
[23]: https://docs.docker.com/engine/userguide/eng-image/multistage-build/#use-multi-stage-builds
[24]: https://twitter.com/markekraus
[25]: https://get-powershellblog.blogspot.no/2017/10/why-pwsh-was-chosen-for-powershell-core.html