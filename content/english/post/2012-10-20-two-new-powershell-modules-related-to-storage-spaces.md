---
title: Two new PowerShell modules related to Storage Spaces
author: Shay Levy
type: post
date: 2012-10-20T13:19:16+00:00
url: /2012/10/20/two-new-powershell-modules-related-to-storage-spaces/
categories:
  - News
tags:
  - News

---
[Bruce Langworthy][1],  Senior Program Manager with the [Storage and FileSystems team at Microsoft][2], just posted [two new PowerShell modules][3] designed to help with the management and diagnosis of Storage Spaces.

This module automates deployment of Storage Spaces and provides intent-based management. For example, instead of using many different cmdlets from the Storage module to create a Storage Space, initialize it, partition it, and format it, all of these tasks are performed via a single cmdlet in Windows PowerShell.

Additionally these cmdlets will work in conjunction with a Failover Cluster using shared SAS storage with Storage Spaces to create a Cluster Shared Volume (CSV) in a single step as well.

Read more and download the module here:

<http://gallery.technet.microsoft.com/scriptcenter/Storage-Spaces-module-for-4be70a0c>

### Storage Spaces Performance Diagnostic

This module can be used to diagnose issues where one or more disks in a Storage Pool are performing abnormally, which are resulting in a Storage Space performing slowly.

Read more, and download this module here:

<http://gallery.technet.microsoft.com/scriptcenter/Storage-Spaces-Performance-9366b756>

<div>
</div>

[1]: https://twitter.com/BruceLangworthy
[2]: http://blogs.msdn.com/san
[3]: http://blogs.msdn.com/b/san/archive/2012/10/19/two-new-modules-related-to-storage-spaces-for-windows-powershell.aspx