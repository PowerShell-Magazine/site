---
title: Build Your Visual Studio Code Development Environment Using PowerShell DSC
author: Ravikanth C
type: post
date: 2016-06-08T16:00:57+00:00
url: /2016/06/08/build-your-visual-studio-code-development-environment-using-powershell-dsc/
views:
  - 16984
post_views_count:
  - 3667
categories:
  - PowerShell
  - VS Code
tags:
  - PowerShell DSC
  - VS Code

---
I keep re-building my lab machines and in the process I rebuild my development virtual machines. One of the items that gets reinstalled all the time is Visual Studio Code editor. Every time I do this, I end up installing VS Code and all the required extensions manually which isn&#8217;t a good use of keystrokes and mouse clicks (if any!). I knew how to install VS Code editor from the command line but I wasn&#8217;t sure about the extensions. So, that is when I reached out to David Wilson asking if there is a way to install these extensions using command line.

{{< tweet 739841414259826688 >}}

David mentioned that this capability was coming to 1.2.0 and by evening I saw tweets from him that 1.2.0 is released. VS Code version 1.2.0 has the command line parameters for managing extensions.

  * `code --list-extensions lists all installed extensions`
  * `code --install-extension installs an extension`
  * `code --uninstall-extension uninstalls an extension`

So, I started writing a DSC resource module to wrap this functionality so that I can use a complete DSC-based method to build my development environment. A couple of hours later, I got the [DSC resource module for VS Code][1]. This is available on [PowerShell Gallery][2] as well.

This module contains a collection of DSC custom resources for installing VS Code and managing VS Code extensions.

At present, this DSC resource module includes 2 resources.

  * [vscodesetup][3] is used to install VS Code editor.
  * [vscodeextention][4] is used to manage VS Code extensions.

Before you can use any of these resources in a configuration script, you must first import the vscode module or a specific resource from this module.

```powershell
Import-DscResource -Module vscode
Import-DscResource -Module vscode -Name vscodesetup
Import-DscResource -Name vscodeextension
```


### Using vscodesetup resource

The _vscodesetup_ resource can be used to install Microsoft Visual Studio Code editor.

![](/images/vscode1.png)

When using this resource, both the _IsSingleInstance_ and the _Path_ must be specified. The _IsSingleInstance_ can only have &#8216;Yes&#8217; as a possible valid value. This is done to ensure that this resource gets used only once in the configuration document. The _Path_ property takes the path to VS Code setup file. This can be downloaded from <https://go.microsoft.com/fwlink/?LinkID=623230>.

```powershell
VSCodeSetup VSCodeSetup {
    IsSingleInstance = 'yes'
    Path = 'C:\temp\vscodesetup.exe'
    PsDscRunAsCredential = $Credential
    Ensure = 'Present'
}
```


The _PsDscRunAsCredential_ is important because VS Code installation process creates the .vscode folder that stores all extensions under the logged-in user&#8217;s homepath. Without this, this folder gets created at the root of system drive. So, using_PsDscRunAsCredential_, you need to pass the current user credentials.

### Using vscodeextension resource

The _vscodeextension_ can be used to install new VS Code extensions from the marketplace. At this point in time, this relies on the command line provided by VS Code but I am exploring other alternatives. Therefore, only VS Code version 1.2.0 onwards is supported for installing VS Code extensions using this resource.

![](/images/vscode2.png)

The only mandatory property in this resource is the _Name_ property. You can use this to provide the name of the VS Code extension. Instead of dividing this into two properties like Publisher and Name, I decided to merge both of them into the _Name_property. Therefore the value to this property must be of the form _Publisher.ExtensionName_. You can find this from the marketplace URL for the extension. Using this method, you can be sure that you are always installing the right extension.

```powershell
vscodeextension PowerShellExtension {
    Name = 'ms-vscode.PowerShell'
    PsDscRunAsCredential = $Credential
    Ensure = 'Present'
    DependsOn = '[vscodesetup]VSCodeSetup'
}
```


Like the _vscodesetup_ resource configuration, this resource requires _PsDscRunAsCredential_ to ensure the extension gets installed for the current user. Make a note that when using credentials in DSC configuration scripts, you must encrypt them using certificates. If the certificates cannot be deployed in a test or development environment, you can use the_[PsDscAllowPlainTextPassword][7]_ attribute in the DSC configuration data. Remember that this is not recommended in production environment.

Here is an example configuration document that installs VS Code and a couple of extensions.

```powershell
$ConfigurationData = @{
    AllNodes = 
    @(
        @{
            NodeName = '*'
            PSDscAllowPlainTextPassword = $true
        },
        @{
            NodeName = 'localhost'
        }
    )
}

Configuration VSCodeConfig {
    param (
        [pscredential] $Credential
    )
    Import-DscResource -ModuleName VSCode

    Node $AllNodes.NodeName {
        VSCodeSetup VSCodeSetup {
            IsSingleInstance = 'yes'
            Path = 'C:\temp\vscodesetup.exe'
            PsDscRunAsCredential = $Credential
            Ensure = 'Present'
        }

        vscodeextension PowerShellExtension {
            Name = 'ms-vscode.PowerShell'
            PsDscRunAsCredential = $Credential
            Ensure = 'Present'
            DependsOn = '[vscodesetup]VSCodeSetup'
        }

        vscodeextension CPPExtension {
            Name = 'ms-vscode.cpptools'
            PsDscRunAsCredential = $Credential
            Ensure = 'Present'
            DependsOn = '[vscodesetup]VSCodeSetup'
        }
    }
}

VSCodeConfig -ConfigurationData $ConfigurationData -Credential (Get-Credential)
```


### [][8]TODO

There are certainly a few things I want to improve in this and also add more resources for customizing VS Code environment. I also want to explore if there is a better way to install extensions instead of using the command line provided with version 1.2.0.

[1]: https://github.com/rchaganti/DSCResources/tree/master/vscode
[2]: https://www.powershellgallery.com/packages/vscode
[3]: https://github.com/rchaganti/DSCResources/tree/master/vscode/DSCResources/vscodesetup
[4]: https://github.com/rchaganti/DSCResources/tree/master/vscode/DSCResources/vscodeextension
[5]: https://github.com/rchaganti/DSCResources/tree/master/vscode#using-vscodesetup-resource
[6]: https://github.com/rchaganti/DSCResources/tree/master/vscode#using-vscodeextension-resource
[7]: http://104.131.21.239/2013/09/26/using-the-credential-attribute-of-dsc-file-resource/
[8]: https://github.com/rchaganti/DSCResources/tree/master/vscode#todo