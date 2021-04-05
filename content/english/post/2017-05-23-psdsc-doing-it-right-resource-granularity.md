---
title: '#PSDSC Doing It Right – Resource Granularity'
author: Ravikanth C
type: post
date: 2017-05-23T16:00:02+00:00
url: /2017/05/23/psdsc-doing-it-right-resource-granularity/
views:
  - 12035
post_views_count:
  - 4401
categories:
  - DevOps
  - PowerShell DSC
tags:
  - DevOps
  - PowerShell DSC

---
This is the first article in the #PSDSC Doing It Right series. This series of articles will introduce you to some of the best practices we have learned through some real-world implementations of PowerShell DSC-based configuration management. Read on!

There are several custom DSC resource modules available either via the official [PowerShell team&#8217;s DSC resource kit][2] or via other community repositories. There are guidelines on [contributing to the official resource kit.][3] This includes a set of requirements for creating [High Quality DSC Resource modules][4] and the [style guidelines][5] that must be followed when creating these High Quality Resource Modules.

These guidelines only discuss how you should structure the DSC resource modules, how and what type of tests you should include, how you must document your module, and what coding style guidelines should be followed, and so on but not how you should design your resource module and what a resource within the module should represent. This is not just about logic in the resource module&#8217;s imperative scripts but what and how those imperative scripts configure. In other words, the granularity of resource configuration should be one of the design considerations when writing resource modules.

Let&#8217;s look at an example to understand this.

Here is the [xVMHyperV][6] resource in [xHyper-V][7] module. This resource has a bunch of properties and supports creation of a VM and some of its related components such as network adapters.

![](/images/dscres1.png)

If I look at the properties listed here, I see a few issues with the way this resource is designed.

  * This resource takes an array of switch names _(SwitchName_ property) and then attaches a network adapter for each switch that is specified in the resource instance configuration. This takes care of adding new adapters in future through the same configuration. However, it fails when you want to remove an adapter. Even if you want to implement that logic in the same resource, it becomes complex.
  * While there is support for multiple network adapters, there is no VLAN ID configuration available in the resource. There is no way we can configure other network adapter settings such as bandwidth settings, DHCP Guard, and other supported configuration settings.
  * This module does a lot of heavy lifting when it comes to virtual hard drive and dynamic memory configurations. However, there is no VHDX Q0S configuration that is possible.

While this resource takes care of certain VM settings, it excludes many of the other VM settings. Adding all this support within the same _xVMHyperV_ resource module will only increase its complexity. Instead, separating out the configuration into multiple smaller resource modules would be a good design.

One of the things that I consider when starting out with developing a new resource module is to first understand the resource itself. Let us look at the VM example again. Here is how I would represent the VM and its related components in a hierarchical manner.

![](/images/dscres2.png)

Once I have this representation, I look at what items in the hierarchy are best suited to be resource on their own. For example, processor in the VM context need not a resource on its own. When we create a new VM, we generally assign number of virtual CPU and that is it. But, the processor settings such as Resource Control, NUMA Topology and so on can be packaged into a different resource so that the VM resource need not worry about having all the logic to manage these special configuration settings. The same thought process applies to VM memory as well. I want to be able to manage the dynamic memory settings only when I need and not package them into the VM resource configuration.

However, when it comes to a network adapter, I want to add or remove network adapters without really touching the VM resource configuration. And, configure other settings for these network adapters when I need them. So, I would create separate resources for network adapters and their associated settings.

Simply put, the more complex your resource becomes, the more complex your tests need to be. Without proper tests for a complex resource, you end up creating something that is substandard. Period.

The granular resource design gives me flexibility while reducing the complexity. You may argue that the configuration documents tend to become very long and difficult to write with so many granular resources just to create a single VM. Yes, if you are doing this only one time, it shouldn&#8217;t matter. But, if you plan to reuse these configurations, composite resources solve this exact problem. I combine long and complex configurations into composite resources and use them very often. These composite resources become my [infrastructure blueprints][8].

Here is how my SimpleVM resource looks like.

![](/images/dscres3.png)

**Note**: This is not yet in my public release of [cHyper-V][9] module. If you want to give this a try, just give a shout and I will be able to invite you to try a few more things along with this.

This SimpleVM resource provides functionality that is good enough to create a VM that has a VHD attached and the required memory and CPU configuration. You can choose to leave the default network adapter or remove it if you want to use the xVMNetworkAdapter to add one or more resources separately.

Using this method for resource design also helped me create resources that are used for different types of components. Consider my [xNetAdapterRDMA][10] resource for an example. This resource is used to enable or disable RDMA on a network adapter. This is a very simple resource. It just flips the RDMA configuration on a network adapter.

![](/images/dscres4.png)

I have also written the [xVMNetworkAdapter][11] resource which adds/removes VM network adapters in the management OS or to the virtual machines. So, when I have to add a VM network adapter that is RDMA capable, I could have implemented the logic to make it so in the xVMNetworkAdapter itself. But, that design, while making the resource complex, would prevent me from using the same functionality with physical adapters. So, if I were to configure physical adapter RDMA settings, I would have ended up writing a different resource or do it some other way. Instead, I chose to separate the RDMA configuration into its own module so that I can share that functionality across different types of network adapters.

To summarize, when designing a resource module, you should look at how and what the resource would do and structure your module in a way that allows flexible resource configurations.

Stay tuned for more in this series.

[2]: https://github.com/PowerShell/DscResources/
[3]: https://github.com/PowerShell/DscResources/blob/master/CONTRIBUTING.md
[4]: https://github.com/PowerShell/DscResources/blob/master/HighQualityModuleGuidelines.md
[5]: https://github.com/PowerShell/DscResources/blob/master/StyleGuidelines.md
[6]: https://github.com/PowerShell/xHyper-V/tree/dev/DSCResources/MSFT_xVMHyperV
[7]: https://github.com/PowerShell/xHyper-V
[8]: http://www.powershellmagazine.com/2017/05/15/infrastructure-blueprints-a-way-to-share-psdsc-configurations/
[9]: https://github.com/rchaganti/DSCResources/tree/master/cHyper-V
[10]: https://github.com/PowerShell/xNetworking/tree/dev/Modules/xNetworking/DSCResources/MSFT_xNetAdapterRDMA
[11]: https://github.com/PowerShell/xHyper-V/tree/dev/DSCResources/MSFT_xVMNetworkAdapter