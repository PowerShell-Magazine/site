---
title: Working with binary files in PowerShell
author: Jan Egil Ring
type: post
date: 2017-05-22T16:00:45+00:00
url: /2017/05/22/working-with-binary-files-in-powershell/
views:
  - 22256
post_views_count:
  - 13018
categories:
  - How To
tags:
  - How To

---
This article is co-authored by Jan Egil Ring and Ø__yvind Kallstad

In this article, we will look at how binary files can be interpreted in PowerShell by looking at a real world challenge as an example.

### The challenge

In System Center Data Protection Manager (DPM), there is an agent installed on protected computers for managing backup and restore operations. If there is a lot of DPM servers in the environment and you do not know what DPM server is protecting an agent, you need a way to find this information on the local machine where the DPM agent is installed on. You might want to look at all the DPM servers, but the agent might be inactive and left over from a decommissioned DPM server.

There aren\`t any official references on this topic besides some guidance in a [forum thread][1] on the TechNet Forums:

_Open an administrative command prompt, then run:_

```
C:\> type C:\Program Files\Microsoft Data Protection Manager\DPM\ActiveOwner*
```

The beginning of the first line returned will be the FQDN in Unicode of the DPM Server owning the agent on the protected server/client

Let&#8217;s try:

![](/images/bin1.png)

This gives us the information we want, but not in a very convenient format. Optimally, we would like to gather this information remotely via PowerShell remoting and get an object back with information about the DPM agent. This could be a function containing version information in addition to the name of the DPM server(s) an agent is attached to.

In PowerShell we can use Get-Content (or its alias type) to get the same information:

![](/images/bin2.png)

At this point, I was thinking that regular expressions might be an appropriate way to solve this challenge. I presented the challenge to my colleague Øyvind, which had experience working with binary files like this in PowerShell.

### Working with binary files

Usually you would want to have some kind of documentation of the file format in question before trying to parse a binary file format. Unfortunately, we couldn&#8217;t find any for this file type. However, it seems from reading the file contents raw, that the information we want is right at the beginning of the file format. If you look at the raw format representation of the string we want to extract, you see that each character has a space between them. This tells us that it&#8217;s a Unicode encoded string.

It&#8217;s useful to use a dedicated hex editor when working with binary file types. In the following screenshot, I&#8217;m using the 010 Editor, and as you can see the editor have confirmed our suspicion that the text is in Unicode format.

![](/images/bin3.png)

What we don&#8217;t know is length of this field, so we must do some guess work. Since this information is referring to a hostname or a domain name, we can assume it&#8217;s not going to contain any spaces. We also hope that there will be at least one space between this field and the next one. Building on these assumptions, we can create a do-while loop that keeps reading bytes until we encounter a byte that when converted to a Unicode string equals to an empty string. The resulting string should be the data that we are after.

Note that since we are reading Unicode-encoded strings, we need to read 2 bytes in each pass.

The way to read data from a binary file is to set up a BinaryReader object. This class has the ReadBytes method that we will use to read bytes from the binary stream.

We also need some way of converting the binary data to something meaningful. Since we already have identified the data as Unicode string, we can use the Unicode.GetString static method in the System.Text.Encoding class in .NET for this.

That&#8217;s all we really need for this particular case. You can find the full code at <https://gist.github.com/janegilring/afec8213d3d14e4f436d0f9d88804f74>

If you are interested in learning more about how to parse data from binary file formats, Øyvind did a talk about this at PowerShell Conference Europe 2017. The video recording of the talk is available [here][2].

[1]: https://social.technet.microsoft.com/Forums/en-US/d2570d6c-3213-410f-85bc-7062cef607b4/dpm-2012-sp1-is-there-a-way-to-find-out-which-dpm-server-an-agent-is-currently-backup-up-to?forum=dataprotectionmanager
[2]: https://www.youtube.com/watch?v=3ilvoOZNDLE