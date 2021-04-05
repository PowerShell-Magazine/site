---
title: PSPublicAPI – Module For Listing Free APIs For Use in Software and Web Development
author: Ravikanth C
type: post
date: 2019-06-18T14:54:14+00:00
url: /2019/06/18/pspublicapi-module-for-listing-free-apis-for-use-in-software-and-web-development/
post_views_count:
  - 3082
views:
  - 3142
categories:
  - Module Spotlight
  - PSPublicAPI
tags:
  - Modules

---
The [Public APIs repository on GitHub](https://github.com/public-apis/public-apis) has a list of free APIs that you can use in software and web development. This is a great resource for finding out if there is a free public API for a specific task at hand. For example, if your application requires weather data, you can take a look at several [free API options available](https://github.com/public-apis/public-apis#weather) and select the one that works for you. I have been following this repository and they have recently added something useful &#8212; a [public API to query for public APIs](https://api.publicapis.org/)! 

I quickly created a new PowerShell module that wraps around the public API for the public APIs!

You can install this [module from the gallery](https://www.powershellgallery.com/packages/PSPublicAPI/)  as well.

```powershell
Install-Module -Name PSPublicAPI -Force
```

There are four commands in this module.

**Get-PSPublicAPICategory** Gets a list of categories for the public API.

**Get-PSPublicAPIHealth** Gets the health state of public API service.

**Get-PSPublicAPIEntry** Gets specific APIs or all API entries from the public API service.

**Get-PSPublicAPIRandomEntry** Gets a random API entry public API service or a random API entry matching a specific criteria.

The commands are pretty much self-explained and you can find the docs for each command [here](https://github.com/rchaganti/PSPublicAPI/tree/master/docs).