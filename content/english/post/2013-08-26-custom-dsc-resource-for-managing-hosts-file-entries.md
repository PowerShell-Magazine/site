---
title: Custom DSC resource for managing hosts file entries
author: Ravikanth C
type: post
date: 2013-08-26T16:00:56+00:00
url: /2013/08/26/custom-dsc-resource-for-managing-hosts-file-entries/
categories:
  - PowerShell DSC
  - Module Spotlight
tags:
  - PowerShell DSC
  - Modules

---
[Desired State Configuration][1] is a new feature added to PowerShell 4.0 and Windows Server 2012 R2. DSC is a platform for deployment and management of configuration data. While there are several built-in DSC resources, it is possible to extend what we can achieve with DSC by creating custom resources.

In this article, I am introducing a new module I created to [manage hosts file as a DSC resource][2]. This custom DSC resource helps you add or remove entries to or from the hosts file in Windows Operating System.

Before I show you how you can use it, go ahead and download this custom DSC resource hosted on [Github][2].

Once downloaded, you can copy the files to C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSDesiredStateConfiguration\PSProviders\HostsFile folder.

To start using the hosts file resource, we need to create a configuration document. A typical configuration document for managing hosts file is shown in the example below.

```
Configuration HostsFileExample {
    Node "SRV2-WS2012R2" {
        HostsFile HostsFileDemo {
            HostName = "testhost100"
            IPAddress = "10.10.10.100"
            Ensure = "Present"
        }
    }
}

HostsFileExample
```

The above example helps you add the hosts file entry on the remote computer named SRV2-WS2012R2. In the above example, setting Ensure=”Absent” will remove the host entry if it exists.

We can apply this configuration by saving the script file, dot-sourcing the script file, and then using Start-DSCConfiguration cmdlet to apply the configuration by providing the MOF file folder location as the value to –Path parameter.

```
PS C:\Demo> .\demo.ps1
 Directory: C:\Demo\HostsFileExample
Mode    LastWriteTime      Length    Name
----    -------------      ------    ----
-a---   8/18/2013 2:53 AM  948       SRV2-WS2012R2.mof

PS C:\Demo> Start-DscConfiguration -Path .\HostsFileExample -Wait -Verbose
```

Now, if we need to extend the above example to add different entries to multiple remote computers, we can do that by adding one more Node script block.


    Configuration HostsFileExample {
        Node "SRV2-WS2012R2" {
            HostsFile HostsFileDemo {
                HostName = "testhost100"
                IPAddress = "10.10.10.100"
                Ensure = "Present"
            }
        }
        Node "SRV2-WS2012R2" {
            HostsFile HostsFileDemo {
                HostName = "testhost120"
                IPAddress = "10.10.10.120"
                Ensure = "Present"
            }
        }
    }
    HostsFileExample
When we dot-source the above script, it will create a MOF file for each node specified in the configuration document. We can, then, simply point Start-DSCConfiguration cmdlet to apply the configuration.

Now, what if we want to add multiple host entries per node in the configuration document? Simple! We add multiple HostsFile resource entries in a Node block. For example,


    Configuration HostsFileExample {
        Node "SRV2-WS2012R2" {
            HostsFile HostsFileDemo {
                HostName = "testhost100"
                IPAddress = "10.10.10.100"
                Ensure = "Present"
            }
        }
        Node "SRV3-WS2012R2" {
            HostsFile HostsFileDemo {
                HostName = "testhost102"
                IPAddress = "10.10.10.102"
                Ensure = "Absent"
            }
        }
       	Node "SRV1-WS2012R2" {
           HostsFile TestHost120 {
               HostName = "testhost120"
               IPAddress = "10.10.10.120"
               Ensure = "Absent"
           }
           
           HostsFile TestHost130 {
               HostName = "testhost130"
               IPAddress = "10.10.10.130"
               Ensure = "Present"
       		}
         }
    }
    
    HostsFileExample
This is it. I have made this custom resource available on [Github DSC repository][3] created by [Steven Murawski][4]. Steve has a [great guide][5] on getting started with creating custom DSC resources which certainly was a starting point for the [HostsFile resource][2]. Feel free to log any [issues or change requests][6]. Also, fork it and update to suit your requirements.

[1]: http://technet.microsoft.com/en-us/library/dn249912.aspx
[2]: https://github.com/dsccommunity/NetworkingDsc/tree/228fcf3111f38e2f22ef19fa73fb2efbe47df25a/DSCResources/MSFT_HostsFile
[3]: https://github.com/dsccommunity
[4]: http://stevenmurawski.com/
[5]: https://github.com/PowerShellOrg/DSC/blob/master/README.md
[6]: https://github.com/PowerShellOrg/DSC/issues?state=open