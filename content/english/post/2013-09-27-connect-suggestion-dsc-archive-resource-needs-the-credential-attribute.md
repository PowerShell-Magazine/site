---
title: 'Connect suggestion: DSC Archive resource needs the Credential attribute'
author: Ravikanth C
type: post
date: 2013-09-27T16:00:12+00:00
url: /2013/09/27/connect-suggestion-dsc-archive-resource-needs-the-credential-attribute/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In an earlier article, I showed you [copying DSC resources][1] and PowerShell modules to remote computers. In that article, I showed a method to authorize remote computers to access the UNC path. For the [File][2] and [Package][3] resources, this is not necessary. These two resources support the [_Credential_ attribute for authenticating][4] to the UNC path. However, the story is different for the Archive resource. It has no _Credential_ attribute which is not only inconsistent but also painful as this forces us to change the computer account permissions for the UNC path or use workarounds such as employing File resource to perform the local copy and then extracting the files using Archive resource. Now, why do we have to perform multiple steps for just extracting files from a simple archive?

Some of you might say that extracting over the wire isn&#8217;t recommended and it is always good to copy the files to a local system before extraction. I don&#8217;t agree. The file copy to local system has more or less the same impact on your network resources as extracting over wire.

So, if you also think that the Archive resource must have the Credential attribute, go ahead and vote for this Connect suggestion: [DSC Archive resource should have the Credential attribute][5].

[1]: /2013/09/02/copying-powershell-modules-and-custom-dsc-resources-using-dsc/
[2]: http://technet.microsoft.com/en-us/library/dn282129.aspx
[3]: http://technet.microsoft.com/en-us/library/dn282132.aspx
[4]: /2013/09/26/using-the-credential-attribute-of-dsc-file-resource/
[5]: https://connect.microsoft.com/PowerShell/feedback/details/802524/dsc-archive-resource-should-have-a-credential-attribute