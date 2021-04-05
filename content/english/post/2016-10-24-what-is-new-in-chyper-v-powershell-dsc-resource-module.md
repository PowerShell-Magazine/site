---
title: What is new in cHyper-V PowerShell DSC Resource Module?
author: Ravikanth C
type: post
date: 2016-10-24T13:14:34+00:00
url: /2016/10/24/what-is-new-in-chyper-v-powershell-dsc-resource-module/
featured_image: /wp-content/uploads/2012/12/Hyper-V.png
views:
  - 14484
post_views_count:
  - 2666
categories:
  - Module Spotlight
  - Hyper-V
  - PowerShell DSC
tags:
  - Hyper-V
  - PowerShell DSC
  - Modules

---
I just finished [testing and publishing][1] the [cHyper-V PowerShell DSC resource][2] module to the PowerShell Gallery. I took some time to make changes to this module&#8211;fix bugs and add new functionality. This module on PowerShell Gallery has over 1,500 downloads across all versions whereas the xHyper-V module has over 8000 downloads.

This means there is certainly significant interest in the cHyper-V module and it is time for me to find ways to merge it with the official HQRM module for Hyper-V at some point in future. I need to work towards that. In preparation towards that goal, I started updating my module in a phased approach.

Phase 1 of the process was complete today. I made the following changes to this module.

  * Removed cNATSwitch resource. This really belongs to [xNetworking][3] module and I will open that PR later next week.
  * Removed cSwitchEmbeddedTeaming and enabled that functionality in cVMSwitch.
  * Added cVMIPAddress for anyone who wants to inject IP addresses into a VM running Windows guest OS using DSC. This is very helpful, at least for me, in building automated labs. More on this later.
  * Added cWaitForVMIntegrationComponent for the same reason as cVMIPAddress.
  * Updated cVMNetworkAdapter to fix bugs and make enhancements based on an [open PR in xHyper-V repository][4]. I will push this update to xHyper-V soon to close that PR.
  * Added a [comprehensive list of examples][5] for each resource in this module.

Moving on to phase 2 of this module development, I will add tests to ensure complete code coverage&#8211;both in terms of unit tests and integration tests. This should be complete by end of this week. So, you will see a minor update of this module in [my GitHub repository][1]. I have these tests running internally after every commit but I just don&#8217;t want to make them public in their current state.

In the final phase of the module preparation to align with the HQRM guidelines, I will open pull requests to xHyper-V module to add all new resources and push updates to the existing resources. This should be complete within next month.

Overall, I am very happy with this phased approach and it is helping me do things at my own pace while enabling me to make progress regularly. cHyper-V will continue to exist for all the experimental Hyper-V DSC resources I continue to create. In fact, I have a bunch of them such as cSimpleVM, cVMCommand, cVMFile, and so on.

Stay tuned for more.

[1]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V
[2]: https://www.powershellgallery.com/packages/cHyper-V/3.0.0.0
[3]: https://github.com/PowerShell/xNetworking
[4]: https://github.com/PowerShell/xHyper-V/pull/53
[5]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V/Examples