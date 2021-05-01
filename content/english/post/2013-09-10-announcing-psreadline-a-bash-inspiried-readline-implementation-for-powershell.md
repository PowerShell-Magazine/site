---
title: Announcing PSReadLine – A bash-inspired readline implementation for PowerShell
author: Ravikanth C
type: post
date: 2013-09-10T06:38:02+00:00
url: /2013/09/10/announcing-psreadline-a-bash-inspiried-readline-implementation-for-powershell/
categories:
  - News
  - Module Spotlight
  - PSReadLine
tags:
  - PSReadLine
  - Modules
  - News

---
[Jason Shirk][1] is back! He just announced the release of his new PowerShell project [PSReadLine][2]. In PowerShell 3.0, a hook was added to replace the command line editing experience in the console and Jason wrote a new module to take advantage of this hook. This module is really meant for PowerShell console and not for ISE.

The following list of features are available in the first release.

  * Syntax coloring
  * Simple syntax error notification
  * A better multi-line experience (both editing and history)
  * Customizable key bindings
  * Cmd and emacs modes (neither are fully implemented yet, but both are usable)
  * Many configuration options
  * Bash style completion (optional in Cmd mode, default in Emacs mode)
  * Emacs yank/kill ring
  * PowerShell token based &#8220;word&#8221; movement and kill

A quick overview of this module and how it can be used has been documented at <https://github.com/lzybkr/PSReadLine/blob/master/README.md>

Go ahead and [download the module][2]. If you are interested, you can even fork it and add your own set of features.

[1]: http://104.131.21.239/2011/11/03/an-interview-with-powershell-expert-jason-shirk/
[2]: https://github.com/lzybkr/PSReadLine