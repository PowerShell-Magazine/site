---
title: Royal TS V3 (with a PowerShell module) released
author: Jan Egil Ring
type: post
date: 2015-03-24T16:00:47+00:00
url: /2015/03/24/royal-ts-v3-with-a-powershell-module-released/
views:
  - 11181
post_views_count:
  - 1543
categories:
  - Royal TS
  - Module Spotlight
tags:
  - Modules
  - Royal TS

---
Back in January 2015, I wrote an [introduction][1] to the new PowerShell module in Royal TS V3, which back then was available as a beta version.

Since then, the [final version][2] of Royal TS V3 has released. I provided some feedback for improvements in the previous article, many of them are [implemented][3] in the final version:

- We have unified Get-RoyalObjects and Get-RoyalObject into one cmdlet
- We have added IntelliSense for some parameters with predefined value
- We have updated our help to be more complete and it is now available via Get-Help
- Additionally, we will have a look into getting the Royal TS PowerShell module into Chocolatey/OneGet and in the PowerShell Gallery.

The path to the Royal TS PowerShell module is not added to $env:PSModulePath yet, but I’m told they are looking into it.

I’m also told that Royal TS V3 has been submitted to Chocolatey, but it’s not approved yet. That’s why we still see the previous version when using OneGet to find it:

![](/images/2015-03-04_RoyalTS_Released.png)

As soon as it’s published you may install it using _Install-Package –Name RoyalTS -MinimumVersion 3_. Until then you can get it from the [Royal TS downloads page][2].

The following cmdlets and parameters has been changed in the Royal TS PowerShell module between the beta and final version:

Get-RoyalObject (!)

Folder (+)

Name (+)

Object (-)

Store (+)

Type (+)

Get-RoyalObjects (-)

_!  > Changed_

_+ > New_

_–  > Removed_

Here is an example of the IntelliSense, which has been added:

![](/images/2015-03-04_RoyalTS_Released_02.png)

I have also updated the script for creating Remote Desktop connections I published as part of my previous article, since there was a breaking change from the beta version. You can find the updated version [here][4].

If you have ideas or suggestions for improvements, you may submit them on the [Royal TS Support portal][5].

[1]: http://104.131.21.239/2015/01/08/introducing-the-royal-ts-powershell-module/
[2]: http://www.royalapplications.com/ts/win/download
[3]: http://104.131.21.239/2015/01/08/introducing-the-royal-ts-powershell-module/#comments
[4]: https://github.com/janegilring/PSCommunity/blob/master/Royal%20TS/Update-RoyalFolder.ps1
[5]: http://support.royalapplications.com/forums/242213-royal-ts-for-windows