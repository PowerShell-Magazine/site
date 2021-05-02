---
title: Boost your productivity with Pester snippets
author: Jakub Jareš
type: post
date: 2015-01-13T17:00:22+00:00
url: /2015/01/13/boost-your-productivity-with-pester-snippets/
views:
  - 11908
post_views_count:
  - 1864
categories:
  - Pester
tags:
  - Pester

---
Authoring [Pester][1] tests is easy as it is, but if you are lucky enough to own a copy of [ISESteroids 2][2] it now became even easier. Pester version 3.3.1 adds code snippets for you to use.

ISESteroids comes with TAB expansion of code snippets which is really easy to use. Actually this is how easy it is:

![](/images/pestersnip.gif)

You just type a shortcut and press TAB to expand it.

There is also little to memorize. The Describe is dsc, the Context is ctx and the assertions use the first letter of each word for shortcut. The only exception is Should Be NullOrEmpty which is just sbn and not sbnoe.

### Installing the snippets

Installing the snippets is easy as well. Update your copy of Pester to the latest version (3.3.1) and import it before importing the latest version of ISESteroids (2.0.13.6).

The easiest way to do that is importing both modules in your ISE profile.

This is how you do that:

![](/images/pestersnip2.png)

  * Expand additional menu to access All tools
  * Open the ISE profile
  * Add calls to Import-Module as shown in picture
  * Restart ISE

As noted by Tobias Weltner, author of ISESteroids: Beginning with the current release 2.0.13.6 loading other modules before ISESteroids should not matter anymore. Loading snippets is still done on startup only though, so you have to load Pester first.

[1]: https://github.com/pester/Pester
[2]: http://www.powertheshell.com/isesteroids2/