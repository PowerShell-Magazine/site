---
itle: Using PowerShell and Web Platform Installer to install Azure PowerShell cmdlets
author: Ravikanth C
type: post
date: 2014-02-27T17:00:16+00:00
url: /2014/02/27/using-powershell-and-web-platform-installer-to-install-azure-powershell-cmdlets/
categories:
  - How To
tags:
  - How To

---
[Web Platform Installer][1] or Web PI is a free tool that makes getting the latest components of the Microsoft Web Platform, including Internet Information Services (IIS), SQL Server Express, .NET Framework and Visual Web Developer easy. The Web PI also makes it easy to install and run the most popular free web applications for blogging, content management and more with the built-in [Windows Web Application Gallery][2]. A lot of stuff that you want to do on Azure requires Web PI. For example, the Azure PowerShell cmdlets can be installed using Web PI.

In this article, I am going to show you how to use [Web PI .NET interfaces][3] in PowerShell.

First of all, we need the Web Platform Installer. You can download and install it from <http://www.microsoft.com/web/downloads/platform.aspx>. Once you have Web PI installed, you can access its API by loading the [Microsoft.Web.PlatformInstaller][3] namespace.

<pre class="brush: powershell; title: ; notranslate" title="">[reflection.assembly]::LoadWithPartialName("Microsoft.Web.PlatformInstaller") | Out-Null
</pre>

The [ProductManager][4] class provides access to the product feeds. We can use this class to list the products available for install using Web PI. We can create an instance of this class using the _New-Object_ cmdlet.

```
$ProductManager = New-Object Microsoft.Web.PlatformInstaller.ProductManager
$ProductManager.Load()
```


We need to load the default feed by calling the _Load()_ method. It is not necessary to specify a URI as an argument here. It takes the value of _DefaultFeed_ property and loads it. Once we load the default feed, we can see a list of products by accessing the _Products_ property.

```
$ProductManager.Products | Select Title, Version, Author | Out-GridView
```

![](/images/webpi.png)

We can filter the list of product by using the <em>ProductId</em> property.

```
$ProductManager.Products | Where-Object { $_.ProductId -like "*PowerShell*" } | Select Title, Version | Out-GridView
```

Let us see an example of using Web PI to install a product. For the purpose of this demonstration, I will select Windows Azure PowerShell.

```
$product = $ProductManager.Products | Where { $_.ProductId -eq "WindowsAzurePowerShell" }
```

![](/images/webpi21.png)

We need an instance of the [InstallerManager][5] class to perform the package install.

```
$InstallManager = New-Object Microsoft.Web.PlatformInstaller.InstallManager
$Language = $ProductManager.GetLanguage("en")
$installertouse = $product.GetInstaller($Language)
```

I have also set the language to English so that I get the right installer for my platform. We now need to get the installer for downloading and installing the package. Note that to be able to install packages, you need administrative privileges. So, if you are installing packages, you need to elevate the console.

```
$installer = New-Object 'System.Collections.Generic.List[Microsoft.Web.PlatformInstaller.Installer]'
$installer.Add($installertouse)
$InstallManager.Load($installer)
```

We have now setup the install manager for the right package. We can download the installer package by using the <em>DownloadInstallerFile()</em> method.

```
$failureReason=$null
foreach ($installerContext in $InstallManager.InstallerContexts) {
    $InstallManager.DownloadInstallerFile($installerContext, [ref]$failureReason)
}
```

Finally, we start the package installation by the StartInstallation() method.

```
$InstallManager.StartInstallation()
```

Once the installation is complete, the StartInstallation() method returns $true if everything is successful. You can review the logs at $Product.Installers[0].LogFiles. Here is the complete script that can install Azure PowerShell cmdlets using PowerShell and Web PI.

```
[reflection.assembly]::LoadWithPartialName("Microsoft.Web.PlatformInstaller") | Out-Null

$ProductManager = New-Object Microsoft.Web.PlatformInstaller.ProductManager
$ProductManager.Load()
$product = $ProductManager.Products | Where { $_.ProductId -eq "WindowsAzurePowerShell" }

$InstallManager = New-Object Microsoft.Web.PlatformInstaller.InstallManager

$Language = $ProductManager.GetLanguage("en")
$installertouse = $product.GetInstaller($Language)

$installer = New-Object 'System.Collections.Generic.List[Microsoft.Web.PlatformInstaller.Installer]'
$installer.Add($installertouse)
$InstallManager.Load($installer)

$failureReason=$null
foreach ($installerContext in $InstallManager.InstallerContexts) {
    $InstallManager.DownloadInstallerFile($installerContext, [ref]$failureReason)
}
$InstallManager.StartInstallation()
```

[1]: http://www.microsoft.com/web/downloads/platform.aspx
[2]: http://www.microsoft.com/web/gallery/ "Windows Web Application Gallery"
[3]: http://msdn.microsoft.com/en-us/library/microsoft.web.platforminstaller(v=vs.90).aspx
[4]: http://msdn.microsoft.com/en-us/library/microsoft.web.platforminstaller.productmanager(v=vs.90).aspx
[5]: http://msdn.microsoft.com/en-us/library/microsoft.web.platforminstaller.installmanager(v=vs.90).aspx