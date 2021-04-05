---
title: 'PowerShell OneGet: Gist-As-A-Package'
author: Doug Finke
type: post
date: 2015-01-14T17:00:39+00:00
url: /2015/01/14/powershell-oneget-gist-as-a-package/
views:
  - 21900
post_views_count:
  - 3341
categories:
  - Package Management
  - OneGet
tags:
  - Package Management
  - OneGet

---
This article will cover a new powerful feature delivered in the [PowerShell 5.0 November Preview][1]. This feature is OneGet. [OneGet][2] is a unified package manager for package management systems. A package manager or package management system is a collection of software tools that automates the process of installing, upgrading, configuring, and removing software packages for a computer&#8217;s operating system in a consistent manner.

### What is OneGet?

[OneGet][2] is a unified package manager for package management systems. Environments like Perl, Python, Ruby, Node.js, and JavaScript have them. For the Windows world there is the NuGet and Chocolatey package management systems. PowerShell OneGet delivers a common set of cmdlets to install/uninstall packages, add/remove/query package repositories, and query a system for the software installed, regardless of the installation technology underneath.

Plus, OneGet is open-sourced. This means you can jump on GitHub, see the code used to implement OneGet, participate in discussions about features, and priorities. But wait, there’s more! You can log issues you hit, and you can even create a copy of the source code, make changes and (if you want) request them to be incorporated into the official release.

OneGet also has an SDK that lets others create providers to interop with other technologies. It seamlessly integrates this layer so you, the PowerShell user, don’t even know you’re looking at different repositories containing any array of source code.

I used the OneGet SDK to create my OneGet Gist package provider.

### Gists

Using Gist is a great way to share code. Whether it&#8217;s a simple snippet or a full app, Gist is a great way to get your point across. And the fact that every Gist can be copied and modified makes it even better.

You can discover gists others have created by going to the [gist home page][3] and clicking All Gists. This will take you to a page of all gists sorted and displayed by time of creation or update. You can also search gists by language with Gist Search. Gist search uses the same search syntax as code search.

When you [register (for free) on GitHub][4], you can create gists under your name. Registering lets others discover and install your scripts using the new PowerShell cmdlets found in the **_OneGet_** module.

### Let’s Get Started

Let’s say you’re searching the Gist repository for some scripts (or a colleague sent you a link to his scripts). Using your browser to navigate to the gist and you have a couple of options to get that code to your machine. You can copy and paste it to a file, to an editor, or you can download it as a zip file (then unzip and edit the unzipped content).

Using the Gist Package Provider plugin for OneGet, all you need to do is use **_Install-Package_**.

#### 2.5 steps to getting it working

I outline the steps and then walk through them.

1 &#8211; Install [PowerShell 5.0 November Preview][1].

2 &#8211; Get the Gist package I wrote, using _Install-Module_ (another new feature, that is part of the [PowerShellGet Package Management Module][5]).

2.5 &#8211; Restart the PowerShell console (this is a one-time requirement).

You’re ready to go.

### PowerShellGet

**_PowerShellGet_** _is a package manager for Windows PowerShell.  More specifically, it is a wrapper around a new Windows component called **OneGet**, and it enables simplified package management of PowerShell modules._

[Check out Kirk Munro’s write-up for a deep dive][5].

PowerShellGet is written by Microsoft and leverages the same OneGet SDK I did to get the job done.

PowerShellGet has a repository, the [PowerShell Gallery][6]. _The PowerShell Gallery is the central repository for Windows PowerShell content. You can find new Windows PowerShell commands or Desired State Configuration (DSC) resources in the Gallery._

On it you’ll find the [Gist OneGet provider][7] and you can also check out another module I publish, [PowerShellHumanizer][8], it is used for manipulating and displaying strings, enums, dates, times, timespans, numbers, and quantities.

### Sanity Check

After you install the November Preview, make sure you have the correct version running, _5.0.9883.0_ (or greater if you’re running the latest Windows 10).

![](/images/oneget1.png)

Now you’re ready to install the OneGet Gist Provider, _Install-Module GistProvider_. You need to run as administrator. If this is your first time running a cmdlet from _PowerShellGet_, you’ll be prompted to install _NuGet_, go ahead and say yes. The [PowerShell Gallery][6] is set as an _untrusted_ repository out of the box because the repository is not curated. So, you’ll be prompted asking your permission to install it, type **_y_**, and press enter.

![](/images/oneget2.png)

Now, restart the PowerShell console. This is a one-time operation because of how OneGet initializes providers and is required only after you install an OneGet package provider.

### Is the Package Provider Ready?

After restarting the PowerShell console, make sure the Gist Provider was installed. Use the _Get-PackageProvider_ to list all of the OneGet package providers available.

![](/images/oneget3.png)

Let’s see the provider in action:

![](/images/oneget4.png)

This sends the list of all my, dfinke, public gists to Out-GridView.

![](/images/oneget5.png)

Typing in _Find-Package_ with no parameters, will not return any Gist package info. Why? Because querying the entire Gist repo of authors and their gists would be in the thousands of thousands, so you need to narrow down to the authors you want to query.

You can search for gists using the GitHub webpage. Here is the URL to query all gists with PowerShell, <https://gist.github.com/search?q=powershell>.

![](/images/oneget6.png)

You can specify an array of authors for the –Source parameter, for example:

![](/images/oneget7.png)

Using the filtering capability of _Out-GridView_, you can see a list of gists by author (Source).

![](/images/oneget8.png)

### How to Install a Gist

I have a gist called CFSBuddy.ps1. While OneGet does not support wildcards (yet) you can pass a string to the **_-Name_** parameter for matching. Here I’m searching across multiple authors:

![](/images/oneget9.png)

When you find the package you want, pipe the results to _Install-Package_. Again, the Gist Provider is marked as _untrusted_, prompting you for approval to continue.

![](/images/oneget10.png)

### Where’s the Gist?

The gist are installed in _$env:LOCALAPPDATA\OneGet\Gist_. It saves the downloaded file based on the name the author of the gist saved it as. After installing the CFSBuddy.ps1 script, I can run it like this.

. _$env:LOCALAPPDATA\OneGet\Gist\CFSBuddy.ps1_

You should see this.

![](/images/oneget11.png)

I built this GUI to make it easier to work with the new _ConvertFrom-String_ cmdlet. _ConvertFrom-String_ lets you parse a file by providing a template that contains examples of the desired output data rather than by writing a (potentially complex) script.

This PowerShell based GUI lets you easily try out how _ConvertFrom-String_ acts on your data. As you type in either the _Data_ textbox or the _Template_ textbox, two things happen. The PowerShell that does the work is automatically generated and put in the _Code_ textbox at the bottom, and the PowerShell is executed putting the output in the _Results_ textbox. This GUI really speeds up how you interact with _ConvertFrom-String_.

Check out this great write-up on [ConvertFrom-String][9].

### Multiple Gists Using a One-Liner

This example searches all gists in the repo for author _dfinke_ that contains the string ‘get’. It returns 9 matching gists.

![](/images/oneget12.png)

Piping that to Install-Package will install all the gists (**NOTE**: I’m using the –Force switch, now OneGet will not to prompt me about the Gist Provider being an untrusted source).

### Another Great Way to Share Your Solutions

Now that you’ve seen how easy it is to discover and install gists. You can [register as a user on GitHub][4], paste and save your scripts, then email, tweet, blog, post to the Facebook PowerShell page for colleagues and the community to try it out, like this:

```powershell
Install-Package –Source dfinke –Name CFSBuddy.ps1 –Force 
```

They paste this into a PowerShell console, press enter and the script is located, downloaded and ready to run. Gists are SRR, scripts ready to run.

### Rate Limits

GitHub has rate limits. If you do not pass in credentials, you can make 60 requests in an hour. After 60 requests in the hour, the mechanism the Gist Provider uses will no longer return packages. If you use credentials, you can make up to 5,000 requests per hour. The OneGet *Package cmdlets support credentials and this will work if you have registered on GitHub.

![](/images/oneget13.png)

### Gist Provider on GitHub

I wrote this provider after attending the Microsoft MVP Summit and I’ve put the code up on [my GitHub Repo][10]. I encourage you to check out how it was done (and [my other PowerShell repos][10]).

If you’re up for it, make a copy and your own changes. Please open issues, ask questions or submit suggestion changes/improvements.

As a side note, GitHub is a social network for building software better, together.

So, if you like the provider, visit GitHub and click the star button (and do it for other projects you like). It’s a great way to collaborate, learn, improve your skills and engage with many different communities.

![](/images/oneget14.png)

## Summary

We’ve covered a lot of features and concepts, including OneGet, PowerShellGet, OneGet Providers (e.g. Gist), GitHub Gist, and only mentioned _ConvertFrom-String_.

OneGet and its providers are the easiest way to get PowerShell modules installed (as well as DSC Resources) and it is not just for PowerShell. Check out the list of desired plugins here <https://github.com/OneGet/oneget/issues/77>. Some are already in development.

As a bonus, OneGet has a DSC Resource too. This means more flexibility for your automated DevOps scenarios.

![](/images/oneget15.png)

Happy scripting!

**_Thank you_** to [June Blender][11] for reviewing the article and the great suggestions.

[1]: https://www.microsoft.com/en-us/download/details.aspx?id=44987
[2]: https://github.com/OneGet/oneget
[3]: https://gist.github.com/
[4]: https://github.com/join
[5]: http://blogs.msdn.com/b/mvpawardprogram/archive/2014/10/06/package-management-for-powershell-modules-with-powershellget.aspx
[6]: https://www.powershellgallery.com/
[7]: https://www.powershellgallery.com/packages?q=gistprovider
[8]: https://www.powershellgallery.com/packages/PowerShellHumanizer/
[9]: http://blogs.msdn.com/b/powershell/archive/2014/10/31/convertfrom-string-example-based-text-parsing.aspx
[10]: https://github.com/dfinke/OneGetGistProvider
[11]: http://www.sapien.com/blog/?s=June+Blender