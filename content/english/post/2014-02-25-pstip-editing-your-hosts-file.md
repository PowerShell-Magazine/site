---
title: '#PSTip Editing your hosts file'
author: Shay Levy
type: post
date: 2014-02-25T19:00:53+00:00
url: /2014/02/25/pstip-editing-your-hosts-file/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Occasionally you need to make changes to the hosts file, either locally or remotely on another machine. There are times when you open the file, but when you try to save it you are presented with the &#8220;save as&#8221; dialog, which suggests that the text editor you are working with was not opened as an administrator. As a result, you are forced to close the file, reopen it with elevated editor, and make your changes again.

For that purpose I keep the _Edit-HostsFile_ function in my profile. It takes care of elevation for me (assuming you have the appropriate permissions), making changes to hosts files has never been so easy.


	function Edit-HostsFile
	{
		param($ComputerName=$env:COMPUTERNAME)
		Start-Process notepad.exe -ArgumentList \\$ComputerName\admin$\System32\drivers\etc\hosts -Verb RunAs
	}
