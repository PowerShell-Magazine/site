---
title: '#PSTip Create an empty folder/file using Desired State Configuration File resource'
author: Ravikanth C
type: post
date: 2013-08-26T18:00:06+00:00
url: /2013/08/26/pstip-create-an-empty-folderfile-using-desired-state-configuration-file-resource/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
**Note**: This tip requires PowerShell 4.0 and Windows Server 2012 R2 or Windows 8.1.

When using the [File resource][1] in DSC, a confusing aspect to beginners is how to create an empty folder. If you look at the definition of this resource and the attributes, it gives an impression that you can only copy files/folders from SourcePath to DestinationPath. Well, that is not entirely accurate. You can also create empty folders and files.

For creating an empty folder, use the following configuration document. All you need is the DestinationPath and Type set to Directory.

```
Configuration FileDemo {
    Node SRV1-WS2012R2 {
        File FileDemo {
            Type = 'Directory'
            DestinationPath = 'C:\TestUser3'
            Ensure = "Present"
        }
    }
}

FileDemo
```

This will create an empty folder!

Now, how do we create an empty file? Simple. We provide the path to the file as DestinationPath and supply an empty string as the value to Contents attribute.

```
Configuration FileDemo {
    Node SRV1-WS2012R2 {
        File FileDemo {
            DestinationPath = 'C:\TestUser3\Test.txt'
            Ensure = "Present"
            Contents = ''
        }
    }
}


FileDemo
```




[1]: http://technet.microsoft.com/en-us/library/dn282129.aspx