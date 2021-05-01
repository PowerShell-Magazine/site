---
title: Copying PowerShell modules and custom DSC resources using DSC
author: Ravikanth C
type: post
date: 2013-09-02T16:00:54+00:00
url: /2013/09/02/copying-powershell-modules-and-custom-dsc-resources-using-dsc/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In an earlier article, I showed you a custom DSC resource I built for [managing hosts file entries][1], but I did not tell you that the custom DSC resource must exist on the remote system at a predefined path. When using push configuration model for configuration management, without copying the custom DSC resource, you cannot really apply any configuration supported by the resource. In DSC, there is also a pull model for configuration management which eliminates the need for you to take care of copying the DSC resources to remote systems. We will save this for a later article.

In this article I will show you how to copy the DSC resources from a network share to a set of remote systems using DSC. Before we get started, we need to create a network share and copy all the custom DSC resources.

![](/images/dsc1.png)

Once we copy all the custom DSC resources, we need to assign permissions to the computer accounts that are target nodes for this configuration. This is required because the [DSC Local Configuration Manager][2] runs as the SYSTEM account and won’t have access to network resources. In my lab setup, I just have three virtual machines running Windows PowerShell 4.0 for all my DSC-related work. I have added all the computer accounts to the share with _Read_ permission. As I’d mentioned, this is required and without this you will receive an access denied error when you try to copy files from a network share to a remote system. Thanks to [Steven Murawski][3] for this tip!

![](/images/dsc2.png)

Now, coming to the actual DSC configuration document, we will use a [File][4] resource to perform a copy of a network share contents to a module path for custom DSC resources.


    Configuration CopyDSCResource {
        param (
            [Parameter(Mandatory=$true)]
            [ValidateNotNullOrEmpty()]
            [String[]]$NodeName,
            [Parameter(Mandatory=$false)]
            [ValidateNotNullOrEmpty()]
            [String]$SourcePath,
    
            [Parameter(Mandatory=$false)]
            [ValidateNotNullOrEmpty()]
            [String]$ModulePath = "$PSHOME\modules\PSDesiredStateConfiguration\PSProviders"
        )
    
        Node $NodeName {
            File DSCResourceFolder {
                SourcePath = $SourcePath
                DestinationPath = $ModulePath
                Recurse = $true
                Type = "Directory"
            }
        }
    }
    
    CopyDSCResource -NodeName SRV2-WS2012R2,SRV3-WS2012R2 -SourcePath "\\10.10.10.101\DSCResources"
Once you customize the above configuration document for your requirements, save it as a .ps1 file. Notice the last line in the script where we are specifying the computer names as arguments to the _–NodeName_ parameter and the _-SourcePath_ where all the custom DSC resources are stored.

We can now build the MOF files by dot-sourcing the configuration document and then apply the configuration using _Start-DscConfiguration_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\demo&gt; .\demo.ps1
PS C:\demo&gt; Start-DscConfiguration -Wait -Verbose -Path .\CopyDSCResource
</pre>

Once we complete applying the configuration using the _Start-DscConfiguration_ cmdlet, we can see the custom resource folder in the network share copied to the specified module path.

![](/images/dsc3.png)

Alright, I must admit this is just one way of copying the files from a network share to a remote system. If you have paid enough attention to the attributes available in the File resource, you will ask me a question about the _Credential_ attribute. Yes, this attribute can be used to specify the credentials to access the network share. The Credential attribute eliminates the need for assigning permissions to computer accounts – what is discussed in this article – to access the network shares when applying DSC configuration. However, the _Credential_ attribute comes with its own baggage. Let us save that for a later post! 🙂

[1]: /2013/08/26/custom-dsc-resource-for-managing-hosts-file-entries/
[2]: http://technet.microsoft.com/en-us/library/dn249922.aspx
[3]: http://www.stevenmurawski.com/
[4]: http://technet.microsoft.com/en-us/library/dn282129.aspx