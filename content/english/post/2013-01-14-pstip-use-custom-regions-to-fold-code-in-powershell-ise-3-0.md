---
title: '#PSTip Use custom regions to fold code in PowerShell ISE 3.0'
author: Jakub Jareš
type: post
date: 2013-01-14T19:00:13+00:00
url: /2013/01/14/pstip-use-custom-regions-to-fold-code-in-powershell-ise-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

The PowerShell ISE 3.0 brought a lot of new features. One of them is the ability to hide parts of the code by folding them. The foldable code is marked by small minus/plus signs and can easily fold and unfold by clicking the sign.

![](/images/image001.png)

In this example the parts of the code were marked for folding automatically, but it is also possible to manually define a foldable region. To mark start of the region use &#8216;#region’ keyword optionally followed by name. To mark the end use simply _#endregion_. In the following example I define one around all the function  <br clear="ALL" />definitions to hide them easily:

![](/images/fyn7e1.png)

It is also possible to add name of the region you are ending after the _#endregion_ comment, but keep in mind the name is in this case just a memo, not part of the syntax. See the following example:

![](/images/image0031.png)

I am trying to end the region ‘one’ before the region ‘two’ ended, but regardless of the name specified the region called two is terminated.

Another thing to keep in mind is the #region and _#endregion_ are case sensitive in the PowerShell ISE; specifying #Region or _#endRegion_ unfortunately won’t work. PowerGUI and PowerShell Plus are more forgiving as they don’t care about the case.