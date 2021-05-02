---
title: New features introduced in Windows Management Framework 5.0 Preview September 2014
author: Jan Egil Ring
type: post
date: 2014-09-12T16:00:26+00:00
url: /2014/09/12/new-features-introduced-in-windows-management-framework-5-0-preview-september-2014/
categories:
  - PowerShell DSC
  - News
tags:
  - PowerShell DSC
  - News

---
Windows Management Framework 5.0 Preview September 2014 was [released][1] on September 4.

An overview of new features added since the May 2014 Preview is listed in the [announcement][1] on the Windows PowerShell Team blog, while additional details are available in the [release notes][2].

There are also some new features not mentioned in the release notes, which we will look at in this article.

### Remote File Editing in PowerShell ISE

This feature allows us to edit remote files via PowerShell remoting from the PowerShell ISE.

To use Remote File Editing, open a new PSSession using `Enter-PSSession` and type `PSEdit <path to a file>`. Here is an example:

![](/images/WMF5_Preview_2014_Sept_PSEdit-1024x175.png)

We can see that the file we&#8217;re editing is actually temporary stored in the local AppData folder:

![](/images/WMF5_Preview_2014_Sept_PSEdit_path.png)

Looking at the contents of the _PSEdit_ function, we can see that the file content is retrieved in byte format before it&#8217;s brought back and forth between the local ISE session and the remote PS session:

![](/images/WMF5_Preview_2014_Sept_PSEdit_definition.png)

This can be a very useful feature for working with remote files, especially against Server Core.

### Throttling in Desired State Configuration

A new –ThrottleLimit parameter is introduced for 10 commands in the PSDesiredStatedConfiguration module:

![](/images/WMF5_Preview_2014_Sept_DSC_ThrottleLimit.png)

This parameter enables control over how many nodes to process concurrently; a very useful feature when working with a large number of DSC clients.

### Start-Transcript support in PowerShell ISE

![](/images/WMF5_Preview_2014_Sept_Start-Transcript_help.png)

Historically the Start-Transcript cmdlet has been supported in the PowerShell Console Host only. If you try to use it in a PowerShell ISE version prior to &#8220;5.0 Preview September 2014&#8221; you will be presented with the message &#8220;This host does not support transcription&#8221;.

Introduced in the Windows Management Framework 5.0 Preview September 2014, Start-Transcript works in the PowerShell ISE:

![](/images/WMF5_Preview_2014_Sept_Start-Transcript.png)

**Improvements in COM object performance**

Major performance improvements for working with COM objects is introduced in this release. PowerShell team member Lee Holmes has published a [video][3] demonstrating the improvement.

[1]: http://blogs.msdn.com/b/powershell/archive/2014/09/04/windows-management-framework-5-0-preview-september-2014-is-now-available.aspx
[2]: http://www.microsoft.com/en-us/download/confirmation.aspx?id=44070&6B49FDFB-8E5B-4B07-BC31-15695C5A2143=1
[3]: https://onedrive.live.com/download.aspx?cid=EF4B329A5EB9EA4D&resid=EF4B329A5EB9EA4D%21114&canary=TxIggArSPdvaHqVg9kWKQy0y8rok%2BEOBbi2GoXIlges%3D2