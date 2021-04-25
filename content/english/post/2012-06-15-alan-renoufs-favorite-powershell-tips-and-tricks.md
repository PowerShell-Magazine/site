---
title: Alan Renouf’s Favorite PowerShell Tips and Tricks
author: Alan Renouf
type: post
date: 2012-06-15T18:00:07+00:00
url: /2012/06/15/alan-renoufs-favorite-powershell-tips-and-tricks/
views:
  - 9404
post_views_count:
  - 1424
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When using anything in life with so many features and items you find useful every day of your life how do you narrow it down to 3?  Well, here is my attempt (by the way, I have hundreds more).

### Integration

This might not be a particular tip or trick and is rather high level but it’s my favorite thing about PowerShell, products which were not written to work together inherently can now use PowerShell as a kind of glue to combine information together.  I’m not just talking about products from separate 3<sup>rd</sup> parties but also products within the same company.

For example, back in the day when I was a system administrator I was always trying to export data from various products and use it elsewhere—perhaps check data in another system before performing an action on an item, Active Directory would create a user, give him a mailbox and even create the home directory for him but what if I wanted to take it one step further, add him to our HR system, provision him a virtual desktop or maybe just let someone know he existed… This was always the hard bit.

Now with PowerShell we can not only manage and use the Microsoft products but also expand it to other third parties who have PowerShell snap-ins and modules or even more, expand it to most things with an Application Programming Interface (API) or COM object.

A great example of this was when in a single script I was able recently to deploy bare metal Cisco UCS blades, configure them from front to end, move over to the VMware PowerShell snap-in (PowerCLI) and install the ESXi hypervisor on these blades. After this I could again configure them, add storage with the multiple vendors who have a PowerShell snap-in/module and then introduce them straight into VMware’s cloud platform to allow tenants to use them.  Now that’s Power!

Bare metal to the cloud in a single script which automated multiple companies’ technologies to perform a repeatable and consistent end task.

### 1 or 1 Million Virtual Machines?

It might seem like a simple thing but I love anything that makes my job easier and PowerShell’s range operator (..) certainly does this.

Range operator allows you to define a range of items very easily. In the past if I wanted to do something a number of times I would have needed to use a loop, I had to remember exactly where the brackets went and how to lay it out, now with range operator this becomes so much easier…

![](/images/AlanTips1.png)

So, as you can see from the above we can easily lay out a range of numbers at the shell, but when does this become useful?

How about x amount of new VMs for testing?

![](/images/AlanTips2.png)

And it doesn’t end there, we can also create a range of characters:

![](/images/AlanTips3.png)

As you can see, range operator is certainly a time-saver. I couldn’t tell you how many VMs I have created with this method or how much time this one trick has saved me.

### Skynet is now live

Sometimes I think PowerShell is often a little too amazing, it scares me how easy things are and how tasks which used to be impossible are now easily achievable, almost as if PowerShell was sent back through time and was about to become self aware!

One of the areas that suprises me is the ability to be able to remote control internet explorer and also web pages. This can be very useful when filling out forms or even just for monitoring web sites to make sure the site is up, or the HTML code is what you expect.  It can also be great for grabbing information from web sites.

I’m not going to re-invent the wheel here—one of the best examples of this I have seen can be found [here][1]. It shows how to easily enter text, select radio buttons and press form buttons to submit the data.

[1]: http://msdn.microsoft.com/en-us/magazine/cc337896.aspx