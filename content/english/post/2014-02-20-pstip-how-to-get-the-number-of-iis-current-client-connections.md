---
title: '#PSTip How to get the number of IIS current client connections'
author: Shay Levy
type: post
date: 2014-02-20T19:00:27+00:00
url: /2014/02/20/pstip-how-to-get-the-number-of-iis-current-client-connections/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Using the _Get-Counter_ cmdlet and the Web Service Current Connections performance counter, you can get the amount of current connections to an IIS server or to one of its web sites. This counter is extremely useful in load balanced environments where you want to make sure that connections are evenly balanced across a group of IIS servers.

The following command gets the total number of connections for the server:

```
Get-Counter -Counter 'web service(_total)\current connections' -ComputerName server1
```

This example gets the number of connections for a specified website of three front end servers of a SharePoint farm:

```
Get-Counter -Counter 'web service(sharepoint - myportal80)\current connections'-ComputerName fe1,fe2,fe3
```